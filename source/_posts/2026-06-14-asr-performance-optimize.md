---
title: 一次asr服务性能优化记录
date: 2026-06-14 14:47:38
tags:
  - 小宇宙
---

最近一直在负责小宇宙asr系统的开发上线，第一版已经上线，实际性能表现在2分半处理完4小时的单集。但是第一版的实现方式是在nodejs里直接执行python transcribe脚本，这就导致了模型会在每次执行脚本的时候重复加载到显存里，浪费资源，不仅如此，因为前置还有语言检测步骤，这一步也需要重复加载whisper模型到显存，为了不让语言检测和transcribe两个步骤重复加载的模型在并发时显存溢出，

所以设置的并发数远远低于预期值。于是为了解决这个问题，我决定对python部分做一次重构，具体实现思路很简单，用FastAPI把python部分变成http服务，在python服务启动时就把asr模型加载好，之后在接口里通过已经加载好的模型直接推理即可。第一版代码很简单：
<!--more-->
```
from fastapi import FastAPI, HTTPException
from fastapi_health import health
from pydantic import BaseModel

from language_detector import LanguageDetector
from transcriber import TranscriberFactory

funasr_transcriber = TranscriberFactory.get_transcriber('fun-asr')
whisper_transcriber = TranscriberFactory.get_transcriber('whisper')
language_detector = LanguageDetector()
app = FastAPI()
app.add_api_route('/health', health([]))
@app.get('/language')
def get_language(file: str):
    try:
        language = language_detector.detect(file)
        return {'data': language}
    except Exception as e:
        raise HTTPException(status_code=500, detail=repr(e))

class TranscribeBody(BaseModel):
    file: str
    model: str
@app.post('/transcribe')
def transcribe(body: TranscribeBody):
    if body.model == 'fun-asr':
        funasr_transcriber.transcribe_segment(body.file)
    else:
        whisper_transcriber.transcribe_segment(body.file)

```


结果在并发调用fun-asr模型的transcribe时，fun-asr模型会报错，我测试了用相同音频调用单次请求，以及调用脚本，都是正常执行。然后我搜了一下关于fun-asr多线程的问题，发现有两个相关issue（[链接1](https://github.com/modelscope/FunASR/issues/2475)，[链接2](https://github.com/modelscope/FunASR/issues/2587)）表示fun-asr模型是不支持多线程并发的，也就意味着我只能选择在每次调用时都加载一次模型。

但这个问题不严重，因为fun-asr模型加载占用显存并不大，观察下来执行一次30分钟的音频，占用1.5g显存，相比whisper加载就占用3.3g要好多了，于是我就把代码改成如下结构了

```
executor = ThreadPoolExecutor(max_workers=int(os.environ['TRANSCRIBE_CONCURRENCY']))
@app.post('/transcribe')
async def transcribe(body: TranscribeBody):
    logger.info(f'transcribe with body {json.dumps(body.model_dump())}')
    try:
        if body.model == 'fun-asr':
            funasr_transcriber = TranscriberFactory.get_transcriber('fun-asr')
            await asyncio.get_event_loop().run_in_executor(
                executor,
                funasr_transcriber.transcribe_segment,
                body.file
            )
        else:
            await asyncio.get_event_loop().run_in_executor(
                executor,
                whisper_transcriber.transcribe_segment,
                body.file
            )

```
主要改动就是用了asyncio的event loop + 线程池，可以让并发请求同时执行transcribe。但是在观察gpu情况时，发现gpu利用率一直上不去，表现为大部分时间都很空闲，偶尔会100%，最后完成时间比直接调用脚本要慢了4倍。

于是我继续问AI+搜索，FastAPI如何并发处理CPU密集型任务。给到的大部分答案都是说用多进程的方式，于是我把`ThreadPool`改成了`ProcessPool` ，改完一开始执行会报错

> RuntimeError: Cannot re-initialize CUDA in forked subprocess. To use CUDA with multiprocessing, you must use the 'spawn' start method

搜索得知CUDA在多进程情况下需要在程序最开头加上

`torch.multiprocessing.set_start_method('spawn')`

加上后可以正常运行了，但又发现了新的问题，首先用时情况和上面还是一样，其次每次调用占用显存从1.5g增加到4.4g，具体原因暂时不清楚，总之这种方式也被排除了。

最后我又尝试把asyncio的event loop换成running loop，同时换回`ThreadPool` ，然后对比whisper并发执行后发现，whisper在模型预加载的情况下，并发执行完全没有时间上的增加，观察GPU的使用率也没有任何问题。于是我大胆猜测fun-asr模型的实现方式可能有些问题，导致不能很好地在多线程环境下并发执行推理。于是最终的解决方案是fun-asr模型还是回退到原来的执行脚本的方式，whisper模型常驻显存使用API调用。

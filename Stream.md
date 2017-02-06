# Node 学习笔记
## Steam
### Types of Streams
* Readable 
* Writable
* Duplex 
* Transform

### Object Mode 

Node.js只能创建`String`和`Buffer`流。想创建读取Javascript对象的流，需要自己实现，此时操作Stream就是在称为`Object mode`的模式下。

### Buffering

`Writable`和`Readable`都将接收到的数据存储在内部(internal buffer)。`Writable`通过`writable._writableState.getBuffer()`接收数据，而`Readable`通过`readable._readableState.buffer`接收。
`Writable`和`Readable`内部存储空间依赖传入的`highWaterMark`。不传入时，`hightWaterMark`设为读入流的字节总数，在`Object mode`下，是对象的总数。

## Writable Streams
### Event
* close
    当流及它使用对象(文件、数据库等)关闭后，触发此事件。并不是所有的`Writable Stream`都会触发此事件。
* dran
    当`stream.write(chunk)`返回`false`,触发此事件。
* error
    当`writing`或`piping`时，发生异常，触发此事件。
* finish
    当'stream.end()'被调用后，触发此事件，所有的数据写入成功。
* pipe
    当调用`stream.pipe()`时触发。
* unpipe
    当调用`stream.unpipe()`时触发。

### Methods
* writable.cork()
    此方法强制把所的数据缓存在内存中。只有在调用`stream.uncork()`和`stream.end()`时才会写入。
    主要用来避免大量小数据的情况，大量小数据时实现`writable._writev()`会得到更好的性能。
* writable.end([chunk][,encoding][,callback])
    代表已经写入完成。`chunk`可选参数，提供了在流关闭之前附加上必要的信息。如果提供了`chunk`参数，`callback`会自动监听`finish`事件。
* writable.setDefaultEncoding(encoding)
    设置默认编码。
* writable.uncork()
    使用`writable.cork()`和`writable.uncork()`时，推荐`process.nextTick()`中调用`writable.uncork()`，这样可`writable.write()`就可以在Node.js的事件循环中调用。（这样性能更好吧可能。）
* writable.write(chunk[,encoding][,callback])
    写入数据方法。当返回`false`时，代表`chunk`已经超出了`highWaterMark`的值，此时流将停止直到触发了`drain`事件。

## Readable Streams
### 两种模式
* flowing: 自动流，as quickly as possible。可以通过`EventEmitter`管理。
* paused: 必须手动调用`stream.read()`来读取。

paused mode -> flowing mode:
    - 添加`data`事件
    - 调用`stream.resume()`方法
    - 调用`stream.pipe()`方法，把流导入`Writable`
flowing mode -> paused mode:
    - 没有`pipe`的情况下，调用`stream.paused`.
    - 有`pipe`的情况下，先去掉所有的`data`事件监听，再通过`stream.unpipe()`去掉所有的`pipe`

### 三种状态
* readable._readableState.flowing = null_
* readable._readableState.flowing = false_
* readable._readableState.flowing = true_

## 选择一种方式
尽量选择一种方式来读取文件。推荐使用`readable.pipe()`，可以满足大多数情况。如果想要更多的控制可以使用`EventEmitter`配合`readable.pause()`或`readable.resume()`。

### Event
* close
    当打开的资源(文件等)关闭后触发。
* data
    当读到到数据时触发。当`stream`通过调用`readable.pipe()`,`readable.resume()`转换为`flowing mode`时触发,当添加处理`data`的`callback`时一样会触发。
* end
    读取完成时触发。
* error
    发生任何意外时都可能触发。
* readable
    当有数据可以从流中读取时触发。也会在当流读到最后，会在`end`之间触发。

## Methods
* readable.isPaused()
    是否在`paused mode`下，外部一般不会直接调用此方法。
* readable.pause()
    此方法会引发在`flowing mode`的流停止触发`data`事件，并退出`flowing mode`。
* readable.pipe(destination[, options])
    导入一个或多个可写流。options可会入`end=Boolean`以确定此流读完后，可写流是否关闭，默认是`true`。
* readable.read([size])
    读入固定的数据。如果`size`未设置，内存中的所有数据将返回。
* readable.resume()
    和`readable.pasue()`对应，恢复到`flowing mode`。
* readable.setEncoding(encoding)
    设置编码。
* readable.unpipe([destination])
    解除导流，不设`destination`全部解除。
* readable.unshift(chunk)
    添加`chunk`数据到读取序列的头。
* readable.wrap(stream)
    V.10前的方法。为兼容才保留。旧式的readable stream写法。

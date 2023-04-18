## 1.通过流读取数据

- 用 Readable 创建对象 readable 后，便得到了一个可读流。
- 如果实现 \_read 方法，就将流连接到一个底层数据源。
- 流通过调用 \_read 向底层请求数据，底层再调用流的 push 方法将需要的数据传递过来。
- 当 readable 连接了数据源后，下游便可以调用 readable.read(n) 向流请求数据，同时监听 readable 的 data 事件来接收取到的数据。
  ![](/public/images/stream-how-data-comes-out.png)

## 2.read(fs:2060,372)

read 方法中的逻辑可用下图表示
![](/public/images/stream-read.png)

## 3.push(fs:2109,197)

- 消耗方调用 read(n)促使流输出数据，而流通过\_read()使底层调用 push 方法将数据传给流。
- 如果流在流动模式下(state.flowing 为 true)输出数据，数据会自发地通过 data 事件输出，不需要消耗方反复调用 read(n)。（fs:268）
- 如果调用 push 方法时缓存为空，则当前数据即为下一个需要的数据。这个数据可能先添加到缓存中，也可能直接输出。
- 执行 read 方法时，在调用 \_read 后，如果从缓存中取到了数据，就以 data 事件输出(fs:482)。
- 所以，如果 \_read 异步调用 push 时发现缓存为空，则意味着当前数据是下一个需要的数据，且不会被 read 方法输出，应当在 push 方法中立即以 data 事件输出(\_stream_readable:268)。

## 4.end 事件

- 在调用完 \_read() 后，read(n)会试着从缓存中取数据(\_stream_readable:459)。
- 如果\_read() 是异步调用 push 方法的，则此时缓存中的数据量不会增多，容易出现数据量不够的现象（\_stream_readable:463）。
- 如果 read(n) 的返回值为 null, 说明这次未能从缓存中取出所需量的数据。此时，消耗方需要等待新的数据到达后再次尝试调用 read 方法（\_stream_readable:280）。
- 在数据到达后，流是通过 readable 事件来通知消耗方的（\_stream_readable:280）.
- 在此种情况下，push 方法如果立即输出数据，接收方直接监听 data 事件即可，否则数据被添加到缓存中，需要触发 readable 事件（\_stream_readable:280）
- 消耗方必须监听这个 readable 事件，再调用 read 方法取得数据。

## 5. doRead

- 流中维护了一个缓存，当缓存中的数据足够多时，调用 read() 不会引起\_read() 的调用，即不需要向底层请求数据。
- 用 doRead 来表示 read(n)是否需要向底层取数据（\_stream_readable:431）
- state.reading 标志上次从底层取数据的操作是否已完成。一旦 push 方法被调用，就会设置为 false, 表示此次 \_read() 结束。
- state.highWaterMark 是给缓存大小设置的一个上限阈值。
- 如果取走 n 个数据后，缓存中保有的数据不足这个量，便会从底层去一次数据(\_stream_readable:431)。

## 6.howMuchToRead

- 用 read(n)去取 n 个数据时，m=howMuchToRead(n)是将缓存中实际获取的数据量（\_stream_readable:346）。
- 可读流是获取底层数据的工具，消耗方通过调用 read()方法向流请求数据，流再从缓存中将数据返回，或以 data 事件输出。
- 如果缓存中数据不够，便会调用 \_read 方法去底层取数据。
- 该方法在拿到底层数据后，调用 push 方法将数据交由流处理（立即输出或存入缓存）。
- 可以结合 readable 事件和 read 方法来将数据全部消耗，这是暂停模式的消耗方法。
- read(0) 只是填充缓存区，并不真正读取
- read() 如果处于流动模式，并且缓存区大小不为空，则返回缓存区第一个 buffer 的长度，否则读取整个缓存，如果读到了数据没有返回值，但是会发射 data 事件，数据也能取到，也就是用来清空缓存区。

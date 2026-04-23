# 通信流程

当你将标签靠近读卡器时，标签会进入等待 REQA（Request，请求）或 WUPA（Wake Up，唤醒）命令的状态。

为了检查附近是否有标签，我们以循环方式发送 REQA 命令。如果标签在附近，它会以 ATQA（Answer to Request，应答请求）响应。

一旦收到响应，我们就选择卡片，卡片会返回其 UID（我们不会深入这个过程中的所有技术细节）。之后，我们认证（authenticate）想要读取或写入的扇区。完成操作后，我们发送 HLTA 命令，将卡片置于 HALT（停止）状态。

注意：一旦卡片进入 HALT 状态，只有 WUPA 命令才能唤醒它，让我们继续执行更多操作。

<a href="./images/mifare-flow.png"><img style="display: block; margin: auto;" alt="MIFARE Memory layout" src="./images/mifare-flow.png"/></a>

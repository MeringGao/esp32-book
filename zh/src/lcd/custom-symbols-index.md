# 符号索引（Symbols Index）

以下是自定义符号及其对应字节数组的列表。如果你设计了一个有趣的符号并想添加到这个列表中，欢迎提交拉取请求（pull request）。请使用这里提供的[自定义字符生成器](./lcd-custom-char-generator.md)以确保一致性。

<style>
    /* .preview-symbol{
        width:50px;
        height:80px;
    } */
    .preview-symbol {
        width: 50px;
        height: 80px;
        display: block;
        margin-left: auto;
        margin-right: auto;
        vertical-align: middle;

    }
</style>
<table>
    <thead>
        <tr>
            <th>标题</th>
            <th>预览</th>
            <th>字节数组</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Heart（心形）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/heart.png" alt="heart"></td>
            <td>[ 0b00000, 0b01010, 0b11111, 0b11111, 0b01110, 0b00100, 0b00000, 0b00000,]</td>
        </tr>
        <tr>
            <td>Lock（锁）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/lock.png" alt="lock"></td>
            <td>[ 0b01110, 0b10001, 0b10001, 0b11111, 0b11011, 0b11011, 0b11011, 0b11111, ]</td>
        </tr>
        <tr>
            <td>Hollow Heart（空心心形）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/hollow-heart.png" alt="hollow-heart"></td>
            <td>[ 0b00000, 0b01010, 0b10101, 0b10001, 0b10001, 0b01010, 0b00100, 0b00000, ]</td>
        </tr>
        <tr>
            <td>Battery（电池）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/battery.png" alt="battery"></td>
            <td>[ 0b01110, 0b11011, 0b10001, 0b10001, 0b10001, 0b11111, 0b11111, 0b11111, ]</td>
        </tr>
        <tr>
            <td>Bus（公交车）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/bus.png" alt="bus"></td>
            <td>[ 0b01110, 0b11111, 0b10001, 0b10001, 0b11111, 0b10101, 0b11111, 0b01010, ]</td>
        </tr>
        <tr>
            <td>Bell（铃铛）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/bell.png" alt="bell"></td>
            <td>[ 0b00100, 0b01110, 0b01110, 0b01110, 0b11111, 0b00000, 0b00100, 0b00000, ]</td>
        </tr>
        <tr>
            <td>Hour Glass（沙漏）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/hour-glass.png" alt="hour glass"></td>
            <td>[ 0b00000, 0b11111, 0b10001, 0b01010, 0b00100, 0b01010, 0b10101, 0b11111, ]</td>
        </tr>
        <tr>
            <td>Charger（充电器）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/charger.png" alt="charger"></td>
            <td>[ 0b01010, 0b01010, 0b11111, 0b10001, 0b10001, 0b01110, 0b00100, 0b00100, ]</td>
        </tr>
        <tr>
            <td>Tick Mark（对勾）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/tick-mark.png" alt="Tick Mark"></td>
            <td>[ 0b00000, 0b00000, 0b00001, 0b00011, 0b10110, 0b11100, 0b01000, 0b00000, ]</td>
        </tr>
        <tr>
            <td>Music Note（音符）</td>
            <td><img class="preview-symbol" src="./images/custom-chars/music-note.png" alt="Music note"></td>
            <td>[ 0b00011, 0b00010, 0b00010, 0b00010, 0b00010, 0b01110, 0b11110, 0b01110, ]</td>
        </tr>
    </tbody>
</table>

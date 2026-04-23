## 热敏电阻的非线性特性

热敏电阻的阻值与温度之间存在非线性（non-linear）关系，这意味着随着温度变化，阻值不会按直线规律变化。热敏电阻的行为可以使用 Steinhart-Hart 方程或 B 方程来描述。

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor-non-linearity.jpg"/>

B 方程使用 B 值进行计算，你可以很容易地在网上找到 B 值。另一方面，Steinhart 方程使用 A、B 和 C 系数。有些制造商会提供这些系数，但你仍然需要自己进行校准并找到它们，因为使用 Steinhart 方程的全部目的就是为了获得精确的温度读数。

在接下来的章节中，我们将详细介绍如何使用 B 方程和 Steinhart-Hart 方程来确定温度。


### 参考资料
- [The B parameter vs. Steinhart-Hart equation](https://blog.meteodrenthe.nl/2022/09/07/the-b-parameter-vs-steinhart-hart-equation/)
- [Characterising Thermistors – A Quick Primer, Beta Value & Steinhart-Hart Coefficients](https://community.element14.com/challenges-projects/design-challenges/experimenting-with-thermistors/b/challenge-blog/posts/blog-3-characterising-thermistors-a-quick-primer-beta-value-steinhart-hart-coefficients)

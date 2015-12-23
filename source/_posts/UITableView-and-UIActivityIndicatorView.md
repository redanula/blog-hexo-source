title: UITableView-and-UIActivityIndicatorView 同页面的设置
date: 2015-11-26 00:01:59
tags: markdown
---

备注，后面再细致研究原因。

---


storyboard 中 UIActivityIndicatorView 要放在tabelview下方（拖动后tableview顶部会多出一片空白，这里有个属性adjust scroll view insets取消也可以去空白，但不建议这样做会产生其他问题。）
tableview y设置为0 重设约束才能保证tableview的顶部不会空白。

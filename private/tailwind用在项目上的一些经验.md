
调试略不方便，比如。一个元素是position: absolute，我希望在chrome中暂时去掉这条熟悉，那么正常逻辑是在chrome样式前，把那项打勾去掉。

在tailwindcss的情况就是，是把absolute这个原子样式都去掉。这也会让整个应用用到absolute原子样式的元素都去掉了absolute，就会导致整个页面错乱。

如果要达到一样的效果，我不得不在element.style里手动写一份样式覆盖。多麻烦。

无法自动换行，也就无法单独注释某个样式



不知道为啥不能实现space-between的效果。
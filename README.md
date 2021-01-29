# 一个简单的五子棋实现方案（不含AI）

同事说想开发一个五子棋游戏，按照自己想法做了一个给他们参考，思路是自己想的，算法可能并不科学，仅供参考，不喜勿喷！

查看效果：[https://chenfeng-github.github.io/Gobang/](https://jannyho.github.io/Gobang/)

思路（这里只帖关键代码，创建DOM、CSS等这些基础技术请自行查看源码）：

* 棋盘是一个二维数组，可以修改代码中的 size改变棋盘大小

```javascript
var size = 10;
```
* 用一个变量记录当前执棋方，1为黑，2为白

```javascript
var current = 1;
```
* 根据数组内容的变化，把数据的当前状态映射到界面上，可以用Canvas，也可以用DOM，我这里使用的是DOM。

```javascript
table.addEventListener('click', function (e) {
    e = e || window.event;
    var t = e.target || e.srcElement;
    /* 在td里面加个div，是为了实现圆形棋子以及棋子的阴影 */
    if (t.nodeName.toLowerCase() === 'div') {
      render(t);
    }
    if (t.nodeName.toLowerCase() === 'td') {
      render(t.querySelector('div'));
    }
  })
// 刷新棋盘
  function render(ele) {
    // 获取当前点击对象中储存的行和列信息，根据这些信息操作二维数组
    var row = ele.getAttribute('data-row')
    var column = ele.getAttribute('data-column')
    // 如果当前点击的坐标值为0，证明该坐标尚未有棋子，winner为0表示尚未有胜利者，游戏正在进行中
    if (arr[row][column] === 0 && winner === 0) {
      // 找到数组中对应的坐标，并把当前棋手的标识添加到数组中(黑为1，白为2)
      arr[row][column] = current
      // 改变棋子颜色
      ele.className = 'bg' + current
      // 判断结果，如果有胜出方，result会返回true，如果暂时没有胜出方，则游戏继续
      if (!result(row, column)) {
        // 把当前棋手改为另一方
        current = current === 1 ? 2 : 1;
        // 改变当前颜色提示
        currentElement.className = 'bg' + current;
        // 改当前棋手提示文字
        resultBoard.innerHTML = player[current] + '执棋'
      };
    }
  }
```

* 每次下棋后，根据当前坐标，计算当前行、当前列、左上到右下、右上到左下这条线上是否有连续5个相同的数字，先判断行、再到列，再到斜线，计算方法为使用正则表达式匹配，具体看代码：

```javascript
function result(row, col) {
	// 列结果字符串，因为行可以通过数组的join方法转为字符串，所以不需要用变量记录
    var i, colResult = '', 
      line1Result = '', // 斜线1结果字符串
      line2Result = '', // 斜线2结果字符串
      x, y; // x，y为计算起始坐标
      
    // 统计当前棋子所在行有没有连续的5个棋子，先转为字符串，再用正则匹配
    // 如：[0,1,1,1,1,1,0,0,0,0] 转换为 '0111110000'
    if (arr[row].join('').match(pattern[current])) {
      return showWinner()
    }

    // 统计当前棋子所在列有没有连续的5个棋子，所在列的算法，就是保持列下标不变，遍历每一行
    for (i = 0; i < size; i++) {
      colResult += '' + arr[i][col]
    }
    if (colResult.match(pattern[current])) {
      return showWinner()
    }

    // 统计左下到右上斜线上有没有连续的5个棋子，先获取起始坐标x，超出数组边界则取边界值
    // 再根据起始x坐标计算出起始y坐标
    // 然后x由高到低，y由低到高递增，超出数组边界则停止循环
    x = Math.min(+row + +col, size - 1)
    y = col - (x - row)
    while (x >= 0 && y < size) {
      line1Result += '' + arr[x][y]
      x--;
      y++;
    }
    if (line1Result.match(pattern[current])) {
      return showWinner()
    }

    // 统计左上到右下斜线上有没有连续的5个棋子
    // 左上到右下的起始坐标获取较简单，x坐标取 row - col 的值，y坐标取 col - row 的值
    // 如果x、y为负数，则取 0
    x = Math.max(row - col, 0)
    y = Math.max(col - row, 0)
    while (x < size && y < size) {
      line2Result += '' + arr[x][y]
      x++;
      y++;
    }

    if (line2Result.match(pattern[current])) {
      return showWinner()
    }
  }
```

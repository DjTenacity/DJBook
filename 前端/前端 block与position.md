## 前端 block与position

 block
值	描述
none	此元素不会被显示。
block	此元素将显示为块级元素，此元素前后会带有换行符。
inline	默认。此元素会被显示为内联元素，元素前后没有换行符。
inline-block	行内块元素。（CSS2.1 新增的值）
list-item	此元素会作为列表显示。
run-in	此元素会根据上下文作为块级元素或内联元素显示。
compact	CSS 中有值 compact，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。
marker	CSS 中有值 marker，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。
table	此元素会作为块级表格来显示（类似 <table>），表格前后带有换行符。
inline-table	此元素会作为内联表格来显示（类似 <table>），表格前后没有换行符。
table-row-group	此元素会作为一个或多个行的分组来显示（类似 <tbody>）。
table-header-group	此元素会作为一个或多个行的分组来显示（类似 <thead>）。
table-footer-group	此元素会作为一个或多个行的分组来显示（类似 <tfoot>）。
table-row	此元素会作为一个表格行显示（类似 <tr>）。
table-column-group	此元素会作为一个或多个列的分组来显示（类似 <colgroup>）。
table-column	此元素会作为一个单元格列显示（类似 <col>）
table-cell	此元素会作为一个表格单元格显示（类似 <td> 和 <th>）
table-caption	此元素会作为一个表格标题显示（类似 <caption>）
inherit	规定应该从父元素继承 display 属性的值。

**************************************
## position属性的relative以及absolute的区别是什么？

 任何元素的默认position的属性值都为static（静态），但我们在布局的时候也会经常用到relative（相对）以及absolute（绝对）这两种属性。

  再分别介绍两种属性之前，我们一定要记住一个重要结论：如果用position来进行布局，父级元素的position属性必须为relative，
而定位于父级内部某个位置的元素，最好用absolute，因为它不受父级元素padding的属性影响，当然也可以用relative，不过到时候计算的时候不要忘记padding的值。

  ### 【absolute：绝对定位】
  默认参照浏览器左上角，配合TOP、RIGHT、BOTTOM、LEFT（以下简称TRBL）进行定位，具有以下属性：
  （1）无TRBL的情况下，参照父级左上角；无父级，参照浏览器左上角；无父级元素，但存在文本，参照最后最后一个文字的右上角为原点但是不断开文字，覆盖与上方。
  （2）如果设定TRBL，并且父级没有position属性（不论是absolute还是relative），都是默认以浏览器左上角开始定位，位置由TRBL决定。
  （3）如果设定TRBL，并且父级有position属性（不论是absolute还是relative），默认以父级左上角为原点开始定位，位置由TRBL决定。
 以上三点我们就可以总结出：若想使用absolute进行定位，那我们就要明确两点：
 第一：设定TRBL

 第二：父级设定position属性

  ### 【relative：相对定位】
  默认参照父级原始点为原始点；如果无父级，以文本的上一个元素的底部为原始点，配合TRBL进行定位；当父级内有padding属性时，参照父级内容区的原始点进行定位。
  （1）不存在TRBL，参照父级左上角；没有腹肌，参照浏览器左上角；没有腹肌元素，但是存在文本，则以文本的底部为原始点进行定位并将文字断开。
  （2）设定TRBL，并且父级没有设定position属性，以父级左上角为原点进行定位
  （3）设定TRBL，并且父级设定position属性，以父级左上角为原点进行定位，但是如果父级有padding属性，那么以内容区域的左上角为原点进行定位。

   综上所述，relative均以父级左上角进行定位，但是受padding的影响。

   这样，我们就可以得知为什么要选用relative定位父级元素，absolute定位内部元素。因为我们布局时用relative后，就不只是用float布局页面了，还可以用TRBL进行布局。
但是如果用absolute来布局页面，搜有的DIV都相对于浏览器的左上角定位，这样适配性会大大下降，用户体验度很低。所以只能用与将某个元素定位于属性为absolute的元素的内部的某个位置。
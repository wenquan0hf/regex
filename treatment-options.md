# 处理选项

上面介绍了几个选项如忽略大小写，处理多行等，这些选项能用来改变处理正则表达式的方式。下面是 .Net 中常用的正则表达式选项：

<table cellspacing="0">
        <caption>表6.常用的处理选项</caption>
        <thead>
            <tr>
                <th scope="col">名称</th>
                <th scope="col">说明</th>
            </tr>

        </thead>
        <tbody>
            <tr>
                <td>IgnoreCase(忽略大小写)</td>
                <td>匹配时不区分大小写。</td>
            </tr>
            <tr>
                <td>Multiline(多行模式)</td>

                <td>更改<span class="code">^</span>和<span class="code">$</span>的含义，使它们分别在任意一行的行首和行尾匹配，而不仅仅在整个字符串的开头和结尾匹配。(在此模式下,<span class="code">$</span>的精确含意是:匹配\n之前的位置以及字符串结束前的位置.) </td>
            </tr>
            <tr>
                <td>Singleline(单行模式)</td>
                <td>更改<span class="code">.</span>的含义，使它与每一个字符匹配（包括换行符\n）。 </td>

            </tr>
            <tr>
                <td>IgnorePatternWhitespace(忽略空白)</td>
                <td>忽略表达式中的非转义空白并启用由<span class="code">#</span>标记的注释。</td>
            </tr>
            <tr>
                <td>ExplicitCapture(显式捕获)</td>
                <td>仅捕获已被显式命名的组。</td>
            </tr>
        </tbody>
    </table>


一个经常被问到的问题是：是不是只能同时使用多行模式和单行模式中的一种？答案是：不是。这两个选项之间没有任何关系，除了它们的名字比较相似（以至于让人感到疑惑）以外。




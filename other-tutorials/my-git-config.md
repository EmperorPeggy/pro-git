# My User-Level Git Config

```
[user]
        email = shihaox@amazon.de
        name = Shihao Xu
[color]
        ui = auto
[core]
        pager = less -FMRiX
        editor = emacs -nw
[push]
        default = simple
[alias]
        dag = log --graph --format='format:%C(yellow)%h%C(reset) %C(blue)\"%an\" <%ae>%C(reset) %C(magenta)%cr%C(reset)%C(auto)%d%C(reset)%n%s' --date-or\
der
        tree = "log --graph --date=short --pretty=format:\"%C(bold yellow)%h%C(red)%d %C(nobold)%C(dim green)%cd %C(nodim white)%s %C(nobold magenta)C: %\
C(ul)%cn%C(noul); A: %C(ul)%an\""
        st = status
        co = checkout
        br = branch
        ci = commit
```

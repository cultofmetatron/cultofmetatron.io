---
Categories:
- Development
- GoLang
Description: ""
Tags:
- Development
- golang
date: 2017-03-04T16:10:53-05:00
menu: blog
title: Project Level go environment with direnv
---

I had a spare weekend to start goin through 
[Writing An Interpreter In Go](https://interpreterbook.com/).
I came into it knowing zero go and was running into the this error.

```

warning: GOPATH set to GOROOT (/Users/cultofmetatron/projects/waiig_code_1.3/01/src/monkey) has no effect
lexer/lexer.go:3:8: cannot find package "monkey/token" in any of:
	/Users/cultofmetatron/projects/waiig_code_1.3/01/src/monkey/src/monkey/token (from $GOROOT)
	($GOPATH not set. For more details see: 'go help gopath')
package ./lexer
	imports runtime: cannot find package "runtime" in any of:
	/Users/cultofmetatron/projects/waiig_code_1.3/01/src/monkey/src/runtime (from $GOROOT)
	($GOPATH not set. For more details see: 'go help gopath')

  ```

The first chapter hited at using [direnv](https://direnv.net/) But didn't really specify
what exactly to put in there.

First you have to install direnv

```

brew install direnv

```

then per [the instructions on the direnv website](https://direnv.net/), I added the following 
to my `.zshrc`

```zsh

eval "$(direnv hook zsh)"

```

Once this was done, I had to go back to the folder containing my src.

At the same level as your `src/` folder, enter the folowing

```zsh

echo "export GOPATH=$(pwd)" > .envrc

```

then type `direnv allow .`

happy go programming!




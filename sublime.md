# Sublime

## Custom build

Tools->Build Systems -> New Build System

```
{
	"cmd": ["plink", "-pw", "__PASSWORD__", "ikuchin@192.168.168.12", "cd ~/src/dev.connme.ru && make"]
}
```

```
{
	"fade_fold_buttons": false,
      "caret_style": "phase",
	"highlight_line": true,
	"default_encoding": "Cyrillic (Windows 1251)",
	"fallback_encoding": "Cyrillic (Windows 1251)",
	"font_size": 8
}
```

* Install Sublime3
* Goto [https://packagecontrol.io/installation](https://packagecontrol.io/installation)
* Copy to clipboard
* Switch to Sublime
* Ctrl+\` -> Paste from clipbuffer
* Wait until Package manager will be installed
* Restart Sublime
* Preference->Settings->User settings

## Package manager

* Install Sublime3
* Goto [https://packagecontrol.io/installation](https://packagecontrol.io/installation)
* Copy to clipboard
* Switch to Sublime
* Ctrl+\` -> Paste from clipbuffer
* Wait until Package manager will be installed
* Restart Sublime
* Preference->Settings->User settings

### HighlightWords

{% embed url="https://packagecontrol.io/packages/HighlightWords" %}

{% hint style="warning" %}
Don’t be confused with WordsHighlight – this is different package
{% endhint %}

### FTPSync

* Ctrl+Shift+P
* Enter inside “Install Package”
* Find “FTPSync”
* Install it

### SublimeLinter

### CSSLinter

### ESLint

#### Installation

{% embed url="https://eslint.org/docs/user-guide/getting-started" %}

* Install node.js ([https://nodejs.org/en/#download](https://nodejs.org/en/#download))
* Install node.js module npm install eslint --save-dev
* Init the module npx eslint –-init
* Test that it works npx eslint yourfile.js
* Ctrl+Shift+P
* Enter inside “Install Package”
* Find “ESLinter”
* Install
* Restart Sublime
* Ctrl+Alt+E

If you are getting error _no module found_ try to `npx link eslint`, this command will create link to eslint folder in your $HOME\_DIRECTORY/npm\_modules

#### Config

Sublime -> Preference -> Package Settings -> ESLint -> Settings - Default

```
  "config_file": "c:/Users/ikuchin/AppData/Roaming/Sublime Text 3/Packages/ESLint/.eslintrc.json"
```

#### Standard/standard-packages (do not use)

[www.standardjs.com](http://www.standardjs.com)

Standard JS name claims that it is actually standard, but it is \_NOT\_. Rules in this project driven by group of team-members, not actual standard.

Do not use JS-linter due to strict rules enforcement.

* Enforces removal of semicolon at the end of line
* Enforces spaces instead of tabs
* Etc….

### CPPCheck

* Google “Sublime CPPCheck” (pick the http://cppcheck.sourceforge.net/ link)
* Install cpp application
* Добавить путь к установленному CPPCheck в environment variable PATH
* Ctrl+Shift+P
* Enter inside “Install Package”
* Find “SublimeLinter-CPPCheck”

{% hint style="warning" %}
Not a SublimeLinter-CPPLint
{% endhint %}

* Install
* Restart Sublime


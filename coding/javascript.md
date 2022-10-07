# JavaScript

## \_this\_ binding lost in module event handler

#### Option 1 (use bind)

Setup:

Inside module you need to create event handler (for example: `click` handler). If use simple way:

```
document.getElementById("123").addEventListener("click", this._myHandler)
```

then after once event fired, inside handler object `this` will point out to DOM-element that triggered event.

![](<../.gitbook/assets/image (3).png>)

If you expect `this` reference to module object, rather than DOM-tag, than event listener mist `bind` correct object to handler.

```
document.getElementById("123").addEventListener("click", this._myHandler.bind(this))
```

Notice `bind(this)` at the end of event-handler function.

#### Option 2 (use closure)

Another option is to save `this` (module variable) and then use it in event handler

```
let mod = this; 
document.getElementById("123").addEventListener("click", function(e) {
	mod._myHandler(e);
});
```

Line (1) - saves module object

Line (3) - calls event handler defined inside module

## Excel / XLS export from JavaScript

Option 1: export to XLS using `<a href="data:application/vnd.ms-excel, ...." download="my_file.xls">` . Basic idea is that Excel is capable to open Web-page and tag is simply saves file containing HTML.&#x20;

{% embed url="https://www.codexworld.com/export-html-table-data-to-excel-using-javascript/" %}

Option 2: export using SheetJS.

{% embed url="https://sheetjs.com/" %}

Code example is here: [https://codepedia.info/javascript-export-html-table-data-to-excel](https://codepedia.info/javascript-export-html-table-data-to-excel)

## ESLinter

As a GitHub action I'm using [super-linter](https://github.com/github/super-linter) to lint JS code. Super-linter has multiple modules to check different types of code (Go / C++ / Ruby / etc....).

Here is the Github action config:

{% code lineNumbers="true" %}
```yaml
      ################################
      # Run Linter against code base #
      ################################
      - name: Lint Code Base
        uses: github/super-linter@latest
        env:
          # --- LOG_LEVEL: DEBUG
          VALIDATE_ALL_CODEBASE: false
          VALIDATE_JAVASCRIPT_ES: true
          DEFAULT_BRANCH: master
          FILTER_REGEX_INCLUDE: html/js/pages/.*\.js
          LINTER_RULES_PATH: .github/actions/linters
          TYPESCRIPT_ES_CONFIG_FILE: .eslintrc.yaml
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
{% endcode %}

Line (8) - check only changed files

Line (9) - check only JavaScript against ES-standard

Line(12) and (13) - specify location of ES linter config file



#### Local installation and integration with IDE&#x20;

Described in Sublime -> ESlint section

#### Docker run

{% code lineNumbers="true" %}
```bash
docker run --workdir /github/workspace --rm \
-e VALIDATE_ALL_CODEBASE=true \
-e VALIDATE_JAVASCRIPT_ES=true \
-e RUN_LOCAL=true \
-e USE_FIND_ALGORITHM=true \
-e JAVASCRIPT_ES_CONFIG_FILE=eslint/.eslintrc.yaml \
-v "c:\docs\src\dev.infomed-stat.ru\http":"/tmp/lint" \
-v "c:\Users\ikuchin\Downloads\111111111":"/action/lib/.automation/eslint" \
ghcr.io/github/super-linter
```
{% endcode %}

Line (2) - says check all files, if it will be missed than it will be checking only changed files. But container has no means to figure out if file has or has not been changed. So no files will be checked.

Line (3)  - JavaScript ES validation only

Line (4) - instructs linter to run locally, otherwise it will look for GITHUB variables.

Line (8) and (6) - Default location of all linter configs is /action/lib/.automation, if mount exactly to this directory then all configs will disappear, which prevents from successful run. Do NOT override this folder. Instead mount sub-directory `eslint` (line (8)) and point out to a new eslint config using env variable in line (6).



.eslintrc.yaml

{% code lineNumbers="true" %}
```yaml
extends: "eslint:recommended"
env:
    browser: true
    es2021: true
    jquery: true

# --- avoid no-undef warning for variables below
globals:
    helpdesk_ticket_list_obj: writable
    Image: writable
    isMobile: writable
    jQuery: writable
    localStorage: writable
    location: writable
    navigator: writable
    Notification: writable
    session_pi: writable
    setTimeout: writable
    sms_confirmation: writable
    system_calls: writable
    system_notifications: writable
    StackTrace: writable
    troubleshooting: writable
    Uint8Array: writable
    DrawUserAvatar: writable
    common_bestbounty: writable
    common_connme: writable
    infomed_stat: writable
    common_timecard: writable

ignorePatterns: 
  - "**/*.js"
  - "!**/js/pages/**/*"

overrides:
  - files: 
      # --- infomed-stat
      - "**/months_model/*.js"
      - "**/survival_analysis/*.js"
      - "**/pd/save2xls.js"
    parserOptions:
        sourceType: module

# --- rules turn on/off
#rules:
#    "no-undef": off

```
{% endcode %}

Lines (31) - (33) configure ignore everything except "pages"-folder.&#x20;

Lines (35) - (42) configure some folders/files as a module.

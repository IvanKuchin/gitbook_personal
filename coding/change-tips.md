# Change tips

### Размер загружаемых картинок в «ленту»

Менять нужно в 2-ух местах

* src/xxx/include/localy.h - FEEDIMAGE\_MAX\_FILE\_SIZE
* httpd/xxx/html/js/pages/news\_feed.js – 2 x maxFileSize

### Размер загружаемой аватарки в профиль

Менять нужно в 2-ух местах

* src/xxx/include/localy.h -AVATAR\_MAX\_FILE\_SIZE
* httpd/xxx/html/js/pages/edit\_profile.js – maxFileSize

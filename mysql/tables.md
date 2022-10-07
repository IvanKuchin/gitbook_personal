# Tables

## Table: action\_types

```
id
categoryID – категория 
title – тип действия
isShowFeed – показывать в ленте или нет 
isShowNotification – показывать в оповещениях или нет 
```

![](../.gitbook/assets/MySQL\_tables\_action\_types.jpg)

## Table: feed

```
id
title – описание (в основном «пустое»)
userId – ccылка на таблицу `user`.`id`
action_type_Id – ссылка на таблицу `action_types`.`id`
actionId – уточняющий параметр
eventTimestamp – дата события
```

{% hint style="warning" %}
Первая запись в таблице (id = 1) должна быть пустой. Поскольку в коде используются конструкции типа SELECT @A:=MAX(`id`)+1 FROM `feed`
{% endhint %}

<table><thead><tr><th data-type="number">Type</th><th>userId</th><th>Place to show</th><th>Description</th><th>actionId</th></tr></thead><tbody><tr><td>1</td><td>employee userID</td><td>feed</td><td>сменил(а) место работы</td><td>users_company.id</td></tr><tr><td>2</td><td>user.id</td><td></td><td>поменял длительность работы</td><td>users_company.id</td></tr><tr><td>3</td><td>user.id</td><td></td><td>сменил должность</td><td>users_company.id</td></tr><tr><td>4</td><td>user.id</td><td></td><td>сменил фамилию</td><td></td></tr><tr><td>5</td><td>user.id</td><td></td><td>сменил имя</td><td></td></tr><tr><td>6</td><td>user.id</td><td></td><td>скрыл имя</td><td></td></tr><tr><td>7</td><td>user.id</td><td></td><td>скрыл фамилию</td><td></td></tr><tr><td>8</td><td>avatar owner</td><td></td><td>поменял(а) фотографию</td><td>users_avatars.id</td></tr><tr><td>9</td><td>avatar owner</td><td></td><td>удалил(а) фотографию</td><td></td></tr><tr><td>10</td><td>avatar owner</td><td></td><td>добавил(а) фотографию</td><td>users_avatars.id</td></tr><tr><td>11</td><td>message owner</td><td>feed</td><td>написал(а)</td><td>feed.id</td></tr><tr><td>12</td><td>message owner</td><td>feed</td><td>написал(а) год назад</td><td>feed.id</td></tr><tr><td>0</td><td></td><td></td><td></td><td></td></tr><tr><td>14</td><td>user.id (who click accept)</td><td>feed</td><td>добавил в друзья</td><td>user.id (friend)</td></tr><tr><td>15</td><td>user.id (remover)</td><td>feed</td><td>удалил из друзей</td><td>user.id (friend)</td></tr><tr><td>16</td><td>user.id (requester)</td><td>feed</td><td>отправил запрос на дружбу</td><td>user.id (friend)</td></tr><tr><td>17</td><td>userID</td><td>feed</td><td>заблокировал аккаунт</td><td></td></tr><tr><td>18</td><td>userID</td><td>feed</td><td>разблокировал аккаунт</td><td></td></tr><tr><td>19</td><td>message owner</td><td>notify</td><td>написал(а) коментарий</td><td><p>If message - feed_message_comment.id </p><p>if book - book.id </p><p>if certification - users_certification.id if university - users_university.id</p></td></tr><tr><td>20</td><td>employee userID</td><td></td><td>описал опыт работы в компании</td><td>users_company.id</td></tr><tr><td>21</td><td>userID</td><td></td><td>изменил CV</td><td></td></tr><tr><td>22</td><td>certified.userID</td><td>feed</td><td>получил(а) сертификацию</td><td>users_certification.id</td></tr><tr><td>23</td><td>course participant.userID</td><td>feed</td><td>прошел курс обучения</td><td>users_courses.id</td></tr><tr><td>24</td><td>certified userID</td><td></td><td>исправил название сертификации</td><td>users_certifications.id</td></tr><tr><td>25</td><td>course participant.userID</td><td></td><td>исправил название курса обучения</td><td>users_courses.id</td></tr><tr><td>26</td><td>certified userID</td><td></td><td>исправил номер присвоенного сертификата</td><td>users_certifications.id</td></tr><tr><td>27</td><td>schoolboy userID</td><td></td><td>добавил обучение в школе</td><td>users_school.id</td></tr><tr><td>28</td><td>university student userID</td><td></td><td>добавил обучение в ВУЗ’е</td><td>users_university.id</td></tr><tr><td>29</td><td>lang learner userID</td><td></td><td>изучил иностранный язык</td><td>users_language.id</td></tr><tr><td>30</td><td>schoolboy.userID</td><td></td><td>переехал в университет в другой город</td><td>users_school.id</td></tr><tr><td>31</td><td>student</td><td></td><td>переехал в университет в другой город</td><td>users_university.id</td></tr><tr><td>32</td><td>schoolboy.userID</td><td></td><td>перевелся в другую школу</td><td>users_school.id</td></tr><tr><td>33</td><td>student</td><td></td><td>перевелся в другой университет</td><td>users_university.id</td></tr><tr><td>34</td><td>lang learner userID</td><td>feed</td><td>улучшил иностранный язык</td><td>users_language.id</td></tr><tr><td>35</td><td>schoolboy.userID</td><td></td><td>изменил(а) время начала обучения в школе</td><td>users_school.id</td></tr><tr><td>36</td><td>schoolboy.userID</td><td></td><td>изменил(а) время окончания обучения в школе</td><td>users_school.id</td></tr><tr><td>37</td><td>student.userID</td><td></td><td>изменил(а) время начала обучения в университете</td><td>users_university.id</td></tr><tr><td>38</td><td>student.userID</td><td></td><td>изменил(а) время окончания обучения в университете</td><td>users_university.id</td></tr><tr><td>39</td><td>student.userID</td><td>feed</td><td>получил(а) новую степень</td><td>users_university.id</td></tr><tr><td>40</td><td>lang learner userID</td><td>feed</td><td>улучшил(а) уровень иностранного языка</td><td>users_language.id</td></tr><tr><td>41</td><td>skill_owner</td><td>feed</td><td>добавил(а) навык</td><td>users_skill.id</td></tr><tr><td>42</td><td>skill_owner</td><td></td><td>изменил сильную сторону</td><td>users_skill.id</td></tr><tr><td>43</td><td>skill_owner</td><td>notify</td><td>потвердил(а) навык</td><td>skill_confirmed.id</td></tr><tr><td>44</td><td>skill_owner</td><td></td><td>уменьшилось количество людей, потвердивших ваш навык</td><td>users_skill.id</td></tr><tr><td>45</td><td>recommending_userID</td><td>notify</td><td>написал(а) рекомендательное письмо</td><td>user_recommendation.id</td></tr><tr><td>46</td><td>recommending_userID</td><td>notify</td><td>удалил(а) вашу рекомендацию</td><td>recommended_userID</td></tr><tr><td>47</td><td>recommending_userID</td><td>notify</td><td>удалил(а) ранее выданную рекомендацию</td><td>recommended_userID</td></tr><tr><td>48</td><td>recommending_userID</td><td>notify</td><td>исправил(а) ранее выданную рекомендацию</td><td>user_recommendation.id</td></tr><tr><td>49</td><td>message owner</td><td>notify</td><td>понравилось ваше сообщение</td><td>feed_message_params.id</td></tr><tr><td>50</td><td>message owner</td><td></td><td>больше не нравится сообщение</td><td>feed_message.id</td></tr><tr><td>51</td><td>employee</td><td></td><td>добавил(а) место работы к трудовому стажу</td><td>users_company.id</td></tr><tr><td>52</td><td>employee</td><td></td><td>поменял(а) название компании в которой работал</td><td>users_company.id</td></tr><tr><td>53</td><td>book_reader</td><td>feed</td><td>оценил книгу (поставил рейтинг)</td><td>users_books.id</td></tr><tr><td>54</td><td>book_reader</td><td>feed</td><td>прочитал книгу</td><td>users_books.id</td></tr><tr><td>55</td><td>bookID <strong>(unused)</strong></td><td>notify</td><td>написал(а) коментарий к книге</td><td>users_books.id</td></tr><tr><td>56</td><td>certified <strong>(unused)</strong></td><td>notify</td><td>написал(а) коментарий к сертификации</td><td>users_certification.id</td></tr><tr><td>57</td><td>course.visitor</td><td>feed</td><td>оценил(а) курс</td><td>users_courses.id</td></tr><tr><td>58</td><td>Friend of userID</td><td>notify</td><td>сегодня день рождения</td><td>user.id (friend)</td></tr><tr><td>59</td><td>Whom to reject userID</td><td>notify</td><td>отказали в должности</td><td>company_candidates.id</td></tr><tr><td>60</td><td>Company admin userID</td><td>notify</td><td>Просит передать владение компанией</td><td>company_posession_request.id</td></tr><tr><td>61</td><td>New company admin user ID</td><td>notify</td><td>Вам были переданы права владения компанией</td><td>company.id</td></tr><tr><td>62</td><td>Requester user.id</td><td>notify</td><td>владелец решил продолжить управлять компанией самостоятельно</td><td>company.id</td></tr><tr><td>63</td><td>User.id</td><td>feed</td><td>подписался на рассылку компании</td><td>company.id</td></tr><tr><td>64</td><td>User.id</td><td>feed</td><td>подписался на группу</td><td>group.id</td></tr><tr><td>65</td><td>User.id</td><td>feed</td><td>Создал группу</td><td>group.id</td></tr><tr><td>66</td><td>Grantor.User.ID</td><td>notify</td><td>Благодарит за подарок</td><td>gift_thanks.id</td></tr><tr><td>67</td><td>New host user.id</td><td>notify</td><td>Вас добавили как хост мероприятия</td><td>event_hosts.id</td></tr><tr><td>68</td><td>New guest user.id</td><td>notify</td><td>Вас пригласили на мероприятие</td><td>event_guests.id</td></tr><tr><td>69</td><td>Event_host user.id</td><td>notify</td><td>Получено согласие о визите на ваше мероприятие</td><td>event_guests.id</td></tr><tr><td>70</td><td>Event_guest user.id</td><td>notify</td><td>Извещение о начале мероприятия</td><td>event.id</td></tr><tr><td>null</td><td></td><td></td><td></td><td></td></tr><tr><td>98</td><td>user.id</td><td>notify</td><td>Общее извещение , показывается то , что написано в title</td><td>From User.id</td></tr><tr><td>99</td><td>user.id</td><td>notify</td><td>Общее извещение , показывается то , что написано в title</td><td>From Company.id</td></tr><tr><td>100</td><td>All SoW parties</td><td>notify</td><td>Добавили задачу к SoW</td><td>Timecard_tasks_assignment.id</td></tr><tr><td>101</td><td>All SoW parties</td><td>notify</td><td>Изменили название заказчика в SoW</td><td>Timecard_customers.id</td></tr><tr><td>102</td><td>All SoW parties</td><td>notify</td><td>Изменили название проекта в SoW</td><td>Timecard_projects.id</td></tr><tr><td>103</td><td>All SoW parties</td><td>notify</td><td>Изменили название задачи в SoW</td><td>Timecard_tasks.id</td></tr><tr><td>104</td><td>All SoW parties</td><td>notify</td><td>Изменили длительность задачи в SoW</td><td>Timecard_tasks_assignment.id</td></tr><tr><td>105</td><td>All SoW parties</td><td>notify</td><td>Удалили одну из задач в SoW</td><td></td></tr><tr><td>106</td><td>All SoW parties</td><td>notify</td><td>Агенство инициировало подписание SoW</td><td>SoW.id</td></tr><tr><td>107</td><td>All SoW parties</td><td>notify</td><td>Subcontractor signed SoW</td><td>SoW.id</td></tr><tr><td>108</td><td>Agency employees</td><td>notify</td><td>Subc company registered</td><td>Subc company.id</td></tr><tr><td>109</td><td>All SoW parties</td><td>notify</td><td>Agency generated agreements</td><td>SoW.id</td></tr><tr><td>110</td><td>Agency employees</td><td>notify</td><td>Approver registered</td><td>user.id</td></tr><tr><td>111</td><td>Agency employees</td><td>notify</td><td>Agency employee registered</td><td>user.id</td></tr><tr><td>112</td><td>Agency employees</td><td>notify</td><td>Subc requested absence</td><td>Absence.id</td></tr><tr><td>113</td><td>Agency employees</td><td>notify</td><td>Subc changed absence</td><td>Absence.id</td></tr><tr><td>null</td><td></td><td></td><td></td><td></td></tr><tr><td>1001</td><td>companyID</td><td>feed</td><td>добавили нового основателя</td><td>company_founder.id</td></tr><tr><td>1002</td><td>companyID</td><td>feed</td><td>добавили нового владельца</td><td>company_owner.id</td></tr><tr><td>1003</td><td>companyID</td><td></td><td>добавили сферу деятелности к профилю компании</td><td>company_industry_ref.id</td></tr><tr><td>1004</td><td>companyID</td><td></td><td>изменили описание компании</td><td></td></tr><tr><td>1005</td><td>companyID</td><td></td><td>изменили web-site</td><td></td></tr><tr><td>1006</td><td>companyID</td><td></td><td>изменили количество сотрудников</td><td></td></tr><tr><td>1007</td><td>companyID</td><td></td><td>изменили дату основания компании</td><td></td></tr><tr><td>1008</td><td>companyID</td><td></td><td>изменили тип владения компанией</td><td></td></tr><tr><td>1009</td><td>companyID</td><td></td><td>обновили логотип компании</td><td></td></tr><tr><td>null</td><td></td><td></td><td></td><td></td></tr><tr><td>2000</td><td>groupID</td><td></td><td>group logo uploaded</td><td></td></tr></tbody></table>



При добавлении нового события необходимо выполнить следующее:

1. Решить где это событие должно отображаться: лента, оповещения или в обоих местах
2. Внести изменения в таблицу action\_types с соответсвующими полями
   1. isShowFeed
   2. isShowNotification
3. Нужна или нет дополнительная категория ? (Table - `action_category`)
4. Добавить событие в этот фаил, таблица с описанием событий
5. system.cpp -> GetUserNotificationSpecificDataByType - добавить создание notificationObj по ID
6. common.js -> navMenu\_userNotification. GetAdditionalTitle – добавить специфичные поля для основной информации

## Table: feed\_images

```
Id
set – набор картинок принадлежащих одному сообщению
tempSet – идентификатор устанавливаемый клиентским браузером для серии картинок
userID – ccылка на таблицу `user`.`id`
folder – /images/feed/
filename – уточняющий параметр
eventTimestamp – дата события
```

{% hint style="warning" %}
Первая запись в таблице (id = 1) должна быть пустой. Поскольку в коде используются конструкции типа SELECT @A:=MAX(`id`)+1 FROM `feed_images`
{% endhint %}

set – используется для того что бы в одном сообщении могло быть больше одной фотографии - «набор картинок». «Набор картинок» привязывается к сообщению этим идентификатором. “set”-идентификатор не может быть использован в качестве идентификатора серии загружаемых изображений, поскольку генерация этого идентификатора происходит после загрузки первой картинки, а идентификатор серии необходим перед началом загрузки, в случае если несколько изображений загружаются одновременно.&#x20;

tempSet – используется на клиентской стороне для идентификации серии загружаемых сообщений (но не для привязки «набора картинок» к сообщению).

1. При открытии модального окна tempSet для пользователя userID обнуляется. Во избежание дубликатов&#x20;
2. PostMessage&#x20;
   1. постит сообщение в ленту&#x20;
   2. генерирует уникальный «set» для набора картинок&#x20;
   3. обнуляет «tempSet»&#x20;
3. Cancel message&#x20;
   1. Удаляет все записи с предустановленным «tempSet»&#x20;
   2. Удаляет фаилы с диска

{% hint style="warning" %}
Эффект «потеряной картинки».&#x20;

В случае когда картинки были загружены на сервер , а сообщение не было ни отправлено в ленту, ни отменено. Картинки будут хранится на сервере и не принадлежать ни одному сообщению.
{% endhint %}

userID – используется только для безопасности&#x20;

1. В момент upload картинки. Проверяется , что пользователю можно заливать только в set который принадлежит этому пользователю. Больше это поле не следует использовать нигде.&#x20;
2. На будущее, если нужно будет редактировать свои картинки.

## Table: feed\_message

```
id
title – заголовок
link – ccылка 
message – тело сообщения
imageSetID – ссылка натаблицу feed_images.set
access – ограничения доступа к сообщению 
public – сообщение показывается у всех в лентах
private – сообщение показывается только в моей ленте
friend – сообщение показывается у меня и у друзей
likes – [2delete]
```

{% hint style="warning" %}
Первая запись в таблице (id = 1) должна быть пустой. Поскольку в коде используются конструкции типа SELECT @A:=MAX(`id`)+1 FROM `feed_messages`
{% endhint %}

## Table: feed\_message\_comment

```
id
messageID – information below
type – information below
userID – пользователь написавший сообщение (ссылка на таблицу users.ID)
comment – коментарий
date – дата написания коментария
```

comment – содержит «сырые данные» любая обработка (например: добавление\
....) должны возникать только в момент рендеринга, то есть на клентском браузере.

| Type    | MessageID        | Description     |
| ------- | ---------------- | --------------- |
| message | feed\_message.id | message comment |
| book    | book.id          | book comment    |
| etc...  |                  |                 |

## Table: feed\_message\_param

```
id
messageID – information below
parameter – information below
userID – пользователь нажавший like (ссылка на таблицу users.ID)
date – дата like event
```

Similar "type" approach as described in [previous section](tables.md#table-feed\_message\_comment).

## Table: company

«Владельцем компаниии» может быть любой, если компания никому не принадлежит. Если владельцем хочет стать новый администратор, а компания уже принадлежит существующему, новый администратор должен отправить запрос. Администрация сайта помогает разрешить споры между влдельцами компании только при наличии уставных документов. (поле isConfirmed)

```
id
admin_userID – id пользователя администрирующего компанию
isConfirmed – администратор подтвердил или нет право владения компанией документально (only for site admin usage)
lastActivity – обвновляется когда меняются детали компании, меняется владелец, постятся сообщения от компании. Может использоваться в случаях необходимости смены владельца в случае долгой неактивности или cyber-squatting. 
```

## Table: users

```
id
email – используется для логина пользователей. 
last_online – используется для presense (обновляется _только_ system.cgi)
```

## Table: users\_company

```
id
user_id – ссылка  на таблицу `users`.`id`
company_id – ccылка на таблицу `company`.`id`
position_title_id – ссылка на таблицу `users_company_position`.`id`
occupation_start – дата начала работы в компании
occupation_finish – дата ухода из компании, если «0000-00-00» означает, что в данный момент работаю на этой должности
current_company (retired by occupation finish) 
	0 – в данный момент не работаю на этой должности
	1 – в данный момент работаю на этой должномти
responsibilities – резюме о работе на этой должности

```

current\_company - <mark style="color:red;">retired by occupation finish</mark>

## Table: users\_friends

```
id
userID – ссылка  на таблицу `users`.`id`
friendID – ccылка на таблицу `users`.`id`
state – ссылка на таблицу `users_company_position`.`id`
date – дата обновления статуса запросов
```

![](../.gitbook/assets/MySQL\_tables\_users\_friends.jpg)

## Table: users\_recommendation

```
id
recommended_userID – ссылка  на таблицу `users`.`id`, тот кому написана рекомендация
recommending_userID – ссылка  на таблицу `users`.`id`, тот кто написал рекомендацию
title - рекомендация
eventTimestamp – дата написания рекомендации
state – состяние
```

![](../.gitbook/assets/MySQL\_tables\_users\_recommendations.jpg)

## Table: session

```
user – user e-mail 
time – используется для контроля сессии и cookie.expiration
```

Есть соблазн переделать user.email -> user.id, что якобы дожно позволить уйти от поиска user.id по user.e-mail, но это невозможно. Поскольку аутентификация пользователей осуществляется по user.email, что вынуждает оставить поиск user.id по user.email в таблице user.

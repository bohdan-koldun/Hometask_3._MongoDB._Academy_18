# Hometask №3. MongoDB.

`mongoimport --db users --collection collectionUsers C:\Users\Kold\Downloads\users.json`

- 2018-05-15T14:32:06.861+0300    connected to: localhost
- 2018-05-15T14:32:08.476+0300    imported 844 documents

### 0. - Накатить бэкап базы

`mongodump --db users --out C:/data/backups/`

- 2018-05-15T14:41:44.883+0300    writing users.collectionUsers to
- 2018-05-15T14:41:45.047+0300    done dumping users.collectionUsers (844 documents)

### 1 - Найти средний возраст людей в системе

`db.collectionUsers.aggregate([{$group: {_id: "avg_age_all_users", age: {$avg: "$age"}}}]);`

- { "_id" : "avg_age_all_users", "age" : 30.38862559241706 }

### 2 - Найти средний возраст в штате Аляска

`db.collectionUsers.aggregate([{$match: {address: {$regex: /Alaska/, $options: "i"}}}, {$group: {_id: "avg_age_alaska_users", age: {$avg: "$age"}}}]);`

- { "_id" : "avg_age_alaska_users", "age" : 31.5 }

### 3 - Начиная от Math.ceil(avg + avg_alaska) (порядковый номер документа в БД ) найти первого человека с другом по имени Деннис

`var cursor_avg = db.collectionUsers.aggregate([{$group: {_id: "avg_age_all_users", age: {$avg: "$age"}}}]);`

`var avg = cursor_avg.hasNext() ? cursor_avg.next() : null;`

`var cursor_avg_alaska = db.collectionUsers.aggregate([{$match: {address: {$regex: /Alaska/, $options: "i"}}}, {$group: {_id:"avg_age_alaska_users", age: {$avg: "$age"}}}]);`

`var avg_alaska = cursor_avg_alaska.hasNext() ? cursor_avg_alaska.next() : null;`

`avg.age;`
- 30.38862559241706

`avg_alaska.age;`
- 31.5

`db.collectionUsers.find({$and: [{friends: {$elemMatch: {name: {$regex: /Dennis /}}}}, {index: {$gt: Math.ceil(avg.age+avg_alaska.age)}}]}, {name: 1, friends: 1, address: 1}).limit(1);`

- { "_id" : ObjectId("5adf3c1544abaca147cdd539"), "name" : "Keller Nixon", "address" : "591 Jamison Lane, Idamay, Minnesota, 3128", "friends" : [ { "id" : 0, "name" : "Clarissa Jones" }, { "id" : 1, "name" : "Macias Riley" }, { "id" : 2, "name" : "Dennis Randolph" } ] }


### 4 - Найти активных людей из того же штата, что и предыдущий человек и посмотреть какой фрукт любят больше всего в этом штате (аггрегация)

`db.collectionUsers.aggregate([{$match: { $and: [{address: {$regex: /Minnesota/, $options: "i"}}, {isActive: true}]}},{$project: {_id: 0, name: 1, address: 1}}]);`
- { "name" : "Jodie English", "address" : "998 Bridgewater Street, Summerfield, Minnesota, 1007" }
- { "name" : "Betsy Haynes", "address" : "480 Ridgewood Place, Hickory, Minnesota, 4541" }
- { "name" : "Riley Salinas", "address" : "965 Indiana Place, Wyoming, Minnesota, 4500" }
- { "name" : "Belinda Montoya", "address" : "174 Louis Place, Glenville, Minnesota, 722" }
- { "name" : "Dickerson Lee", "address" : "448 Columbia Place, Ticonderoga, Minnesota, 8183" }
- { "name" : "Erickson Avery", "address" : "969 Fairview Place, Wikieup, Minnesota, 387" }

`db.collectionUsers.aggregate([{$match: { $and: [{address: {$regex: /Minnesota/, $options: "i"}}, {isActive: true}]}}, {$group: {_id: "$favoriteFruit", count: {$sum: 1}}}, {$sort: {count: -1}}, {$limit: 1}]);`
- { "_id" : "apple", "count" : 4 }



### 5 - Найти саммого раннего зарегистрировавшегося пользователя с таким любимым фруктом

`db.collectionUsers.aggregate([{$match: {favoriteFruit: "apple"}}, {$project: {name: 1, registered: 1, favoriteFruit: 1}}, {$sort: {registered: 1}}, {$limit: 1}]);`

- { "_id" : ObjectId("5adf3c1544abaca147cdd568"), "name" : "Magdalena Compton", "registered" : "2014-01-02T10:16:56 -02:00", "favoriteFruit" : "apple", "features" : "first apple eater" }



### 6 - Добавить этому пользовелю свойтво: { features: 'first apple eater' }

`db.collectionUsers.update({_id: ObjectId("5adf3c1544abaca147cdd568")}, {$set: {features: 'first apple eater'}});`
- WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
- { "_id" : ObjectId("5adf3c1544abaca147cdd568"), "name" : "Magdalena Compton", "registered" : "2014-01-02T10:16:56 -02:00", "favoriteFruit" : "apple", "features" : "first apple eater" }


### 7 - Удалить всех любителей клубники (написать количество удаленных пользователей)

`db.collectionUsers.remove({favoriteFruit: {$eq: "strawberry"}});`
 - WriteResult({ "nRemoved" : 253 });


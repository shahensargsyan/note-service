



``` bash
RUN docker exec  note-db mysql -u root -proot0 -e "drop schema laravel_vue;"

RUN docker exec  note-db mysql -u root -proot0 -e "create schema laravel_vue;"
```

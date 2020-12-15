# MYSQL Query VS LARAVEL QUERY BUILDER

~~~~sql
select * from actor where last_name = 'berry' 
~~~~

```php
# use DB
$result = DB::table('actor')->where('last_name','=','berry')->get()
```

```php
$result = DB::table('actor')->where('last_name','berry')->get()
```
#### Closure 
```php
$result = DB::table('actor')->where(function ($query) {
                $query->where('last_name','berry');
                })->get()
```

#### Condition where is matches both conditions
```php
$result = DB::table('actor')->where([
                ['last_name','berry'],
                ['first_name','joe']
                ])->get()
```
```php
$users = DB::table('users')->where([
            ['status', '=', '1'],
            ['subscribed', '<>', '1'],
            ])->get();
```

#### Aggregate 
~~~~sql
select last_name,count(*) as actor_count from actor group by last_name order by actor_count desc
~~~~

```php
$result = DB::table('actor)
            ->select(['last_name',DB::raw('count(*) as actor_count)])
            ->groupby('actor_count','desc')
            ->get();
```

```php
$users = DB::table('users')
             ->select(DB::raw('count(*) as user_count, status'))
             ->where('status', '<>', 1)
             ->groupBy('status')
             ->get();
```
```php
$users = DB::table('users')
           ->whereBetween('votes', [1, 100])
           ->limit(10);
           ->get();
```
```php
users = DB::table('users')
            ->whereNotBetween('votes', [1, 100])
            ->get();
```
```php
$users = DB::table('users')
            ->whereIn('id', [1, 2, 3])
            ->get();
```
#### Joins
~~~~sql
select s.`id`,s.`name`,s.`email`,addr.`address`,addr.`district`,addr.`postal_code`,c.`city`,con.`country` from staff as s left join address as addr on s.`address_id` = addr.`address_id` left join city as c on addr.`city_id` =c.`city_id` left join country as con on c.`country_id` = con.`country_id` oder by id
~~~~

```php
$result = DB::table('staff as s')
            ->select([
                's.id','s.id','s.name','s.email','addr.address','addr.district','addr.postal_code','c.city','con.country'
            ])
            ->leftJoin('address as addr','s.address_id','=','addr.address_id')
            ->leftJoin('city as c','addr.city_id','=','c.city_id')
            ->leftJoin('country as con','c.country_id','=','con.country_id')
            ->oderBy('id')
            ->get()

```


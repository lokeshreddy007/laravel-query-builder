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
select s.`id`,s.`name`,s.`email`,
    addr.`address`,addr.`district`,addr.`postal_code`,
    c.`city`,con.`country` 
    from 
        staff as s left join address as addr 
        on s.`address_id` = addr.`address_id` 
        left join city as c on addr.`city_id` =c.`city_id` 
        left join country as con on c.`country_id` = con.`country_id` 
        order by id
~~~~

```php
$result = DB::table('staff as s')
            ->select([
                's.id','s.id','s.name','s.email',
                'addr.address','addr.district','addr.postal_code',
                'c.city','con.country'
            ])
            ->leftJoin('address as addr','s.address_id','=','addr.address_id')
            ->leftJoin('city as c','addr.city_id','=','c.city_id')
            ->leftJoin('country as con','c.country_id','=','con.country_id')
            ->oderBy('id')
            ->get()
```
#### Sub Queries

###### writing a quuery to display each etore id city country and sales they made
###### store -> left join -> address
###### address -> inner join  ->city
###### city ->inner join --. country  ====>  //store_details

###### customer - > inner join -->payment  ===>payment_details
~~~~sql
select store_details.*,payment_details.sales
from (
    select sto.store_id,city.city,coun.country
    from store as sto
    left join address as addr
    on sto.address_id = addr.address_id
    join city
    on addr.city_id = city_id
    join country as coun
    on city.country_id = coun.country_id
) as store_details
Inner Join (
    select cus.store_id,sum(pay.amount) as sales
    from customer as cus
    join payments as pay
    on cus.custmoser_id = pay.customer_id
    group by cus.store_id
) as payment_details
on store_details.store_id = payment_details.store_id
order by store_details.store_id
~~~~

### Method 1
```php

# query 1
$store_details= DB::query()
                ->select(['sto.store_id','city.city','coun.country'])
                ->from('store as sto')
                ->leftJoin('address as addr','sto.address_id','=','addr.address_id')
                ->join('city','addr.city_id','=','city_id')
                ->join('country as coun','city.country_id','=','coun.country_id')
# query 2
$payment_details = DB::query()
                ->select(['cus.store_id',DB::raw('sum(pay.amount) as sales')])
                ->from('customer as cus')
                ->join('payments as pay','cus.custmoser_id','=','pay.customer_id')
                ->groupBy('cus.store_id')

# Final Query  combination of both query
$result = DB::query()
        ->select('store_details.*','payemnt_details.sales')
        ->fromSub($store_details,'store_details')
        ->joinSub($payemnt_details,'payemnt_details','store_details.store_id','=','payemnt_details.store_id')
        ->get()

### TO get SQL query from Query builder
$result = DB::query()
        ->select('store_details.*','payemnt_details.sales')
        ->fromSub($store_details,'store_details')
        ->joinSub($payemnt_details,'payemnt_details','store_details.store_id','=','payemnt_details.store_id')
        ->toSql()        

```

### Method 2

```php
$result = DB::query()
        ->select('store_details.*','payemnt_details.sales')
        ->fromSub(function($query) {
            $query->select(['sto.store_id','city.city','coun.country'])
                ->from('store as sto')
                ->leftJoin('address as addr','sto.address_id','=','addr.address_id')
                ->join('city','addr.city_id','=','city_id')
                ->join('country as coun','city.country_id','=','coun.country_id')
        },'store_details')
        ->joinSub(function($query){
            $query->select(['cus.store_id',DB::raw('sum(pay.amount) as sales')])
                ->from('customer as cus')
                ->join('payments as pay','cus.custmoser_id','=','pay.customer_id')
                ->groupBy('cus.store_id')
        },'payemnt_details','store_details.store_id','=','payemnt_details.store_id')
        ->get()  
```
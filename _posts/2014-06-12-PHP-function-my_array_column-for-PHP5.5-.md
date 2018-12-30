---
layout:     post
title:      PHP function my_array_column for PHP5.5-
subtitle:    This article discusses the merits of allowing method calls on primitive PHP types, like strings and arrays.
date:       2014-06-12
author:     Longniao
header-img: img/post-php-banner.jpg
catalog: true
tags:
    - PHP
    - array
---

PHP function array_column() returns the values from a single column of the array, identified by the column_key, bug it can only used in PHP version 5.5+.

What can I do in my PHP 5.2 VPS? It's piece of cake.

Lets start:
-----------

{% highlight php startinline %}

    function my_array_column($array, $colName){

        $results=array();
        if(!is_array($array)) return $results;

        if (function_exists('array_column')) {
            return array_column($array, $colName);
        }   

        foreach($array as $child){
            if(!is_array($child)) continue;
            if(array_key_exists($colName, $child)){
                $results[] = $child[$colName];
            }   
        }   
        return $results;
    }

{% endhighlight %}

How to use it:
--------------

{% highlight php startinline %}

$data = array(
    array(
        'id' => 2135,
        'first_name' => 'John',
        'last_name' => 'Doe'
    ),
    array(
        'id' => 3245,
        'first_name' => 'Sally',
        'last_name' => 'Smith'
    ),
    array(
        'id' => 5342,
        'first_name' => 'Jane',
        'last_name' => 'Jones'
    ),
    array(
        'id' => 5623,
        'first_name' => 'Peter',
        'last_name' => 'Doe'
    )
);

$result = my_array_column($data, 'id');

{% endhighlight %}

So, It's very easy.

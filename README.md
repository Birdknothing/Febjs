# Febjs
Functional programming alike javascript library

## Usage

```html
     <script src="http://static.shaofeibo.cn/feb.min.js"></script>
```

## Concept

This open source javascript library is mean to solve async callback hell problem and improve programming experience.
It provides the "production line" concept to solve a problem and deliver the only-one scope bewtween functions.

## Features

1.The only-one scope is an object having basic propery "val" and "err" ({ val:"", err:"" }).
2.Any error will automatically stop the temporary function node and makes the function line jump to the end.
3.Error-breaker can be manually triggered by set some value to err.
4.Any error can be captured by the next function node to decide if to end the function line or continue.

## Rules

1.The properties "val" and "err" are reserved in the only-one scope and can be only used to set default value (or function) and error.
2.All async methods' success-callback and failure-callback function have to be at the end of the parameters.
3.Success-callback and failure-callback are always required in both async function and sync function.
4.Function lines carrying only-one scope with val whose value type is not Function should always return the scope.

## Methods

1. Object.prototype.bei (pinyin of Chinese word "被")
2. Object.prototype.cuo (pinyin of Chinese word "错")
3. global.then 
4. global.assign
5. global.turn

## Examples

1. The only-one scope deliver value:

```javascript

     function sync_add_2(o){
        o.val += 2;
        console.log("add the value by 2");
        return o;
    }
    
    function sync_minus_1(o){
        o.val -= 1;
        console.log("minus the value by 1");
        return o;
    }
    
    function sync_print_val(o){
        console.log("value is :",o.val);
        return o;
    }
    
    ;({val:1})
    .bei(sync_print_val)
    .bei(sync_add_2)
    .bei(sync_minus_1)
    .bei(sync_print_val);
    
    // value is : 1
    // add the value by 2
    // minus the value by 1
    // value is : 2
    
```

2. The only-one scope deliver function:

```javascript
  
    // success-callback and failure-callback are required in sync function as triggers too
    function sync_print_fun(o,scb,fcb){
            console.log("value is :",o.val);
            // success-callback will move the function line to the next node
            scb && scb();
            // failure-callback will stop the function line immediately
            // fcb();
    }
    function async_add_2(o,scb,fcb){
        setTimeout(() => {
            console.log("add value by 2");
            o.val += 2;
            scb && scb();
        }, 1000);
    }
    function async_minus_1(o,scb,fcb){
        setTimeout(() => {
            console.log('minus value by 1');
            o.val -= 1;
            scb && scb();
        }, 1000);
    }

    const o = {val:3};
    
    ;({})
    .bei(then(sync_print_fun,o))
    .bei(then(async_add_2,o))
    .bei(then(sync_print_fun,o))
    .bei(then(async_minus_1,o))
    .bei(then(sync_print_fun,o))
    .val();
    
    // value is : 3
    // add value by 2
    // value is : 5
    // minus value by 1
    // value is : 4
    
```

3. Manipulate the function line when errors occur

```javascript

   function sync_print_val(o){
        console.log("value is :",o.val);
        return o;
    }
    function sync_trigger_error(o){
        o.err = "new error";
        return o;
    }
    function sync_catch_error_and_continue(o){
        o.err = undefined;
        console.log("remove error and continue");
        return o;
    }
    
    /* 
      prototype method "cuo" will catch the error occurring in the previous function node 
     */
    
    ;({val:1}).bei(sync_trigger_error).bei(sync_print_val);
    // !!! error is :  some error
    ;({val:1}).bei(sync_trigger_error).cuo(sync_print_val);
    // value is : 1
    
    
    /*
      catch the previous err and set it to undefined will continue the function line
     */
    
    ;({val:1}).bei(sync_trigger_error).cuo(sync_print_val).bei(sync_print_val);
    // value is : 1
    // !!! error is :  some error
    
    ;({val:1}).bei(sync_trigger_error).cuo(sync_catch_error_and_continue).bei(sync_print_val);
    // remove error and continue
    // value is : 1
    
```

4. Other usage:

```javascript

   ;({val:1}).bei(assign({x:1})).bei(console.log);
   // {val: 1, x: 1}
   
   ;({val:1}).bei(turn(x => x+1)).bei(console.log);
   // {val: 2}
   
```

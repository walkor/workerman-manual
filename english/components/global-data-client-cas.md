# cas
**``` (Requires Workerman version >= 3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Atomic replacement, replace ```$old_value``` with ```$new_value```.
The value can be written only if the value corresponding to the key has not been modified by other clients since the last time it was fetched by the current client.

## Parameters

 ``` $key ```

The key. (For example, ```$global->abc```, where ```abc``` is the key)

 ``` $old_value ```

Old data

 ``` $new_value ```

New data

## Return Value
Returns true if the replacement is successful, otherwise returns false.

## Note:
When multiple processes operate on the same shared variable, concurrent issues need to be considered.

For example, processes A and B both want to add a member to a user list.
At a certain point, the user list for both processes A and B is ```$global->user_list = array(1,2,3)```.
Process A operates on the ```$global->user_list``` variable and adds a user 4.
Process B operates on the ```$global->user_list``` variable and adds a user 5.
Process A sets the variable ```$global->user_list = array(1,2,3,4)``` successfully.
Process B sets the variable ```$global->user_list = array(1,2,3,5)``` successfully.
At this point, the variable set by process B overrides the one set by process A, leading to data loss.

The above scenario is a result of non-atomic operations for reading and setting, which leads to concurrent issues. To address this concurrent issue, the cas atomic replacement interface can be used. 
Before changing a value, the cas interface checks whether the value has been modified by other processes based on ```$old_value```. If it has been modified, it does not perform the replacement and returns false; otherwise, it performs the replacement and returns true. See the example below.

 **Note:** 
In some cases, concurrent overwriting of shared data is not a problem, such as in a bidding system for the current highest bid on an item or the current inventory of a product.

## Example

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Initialize the list
$global->user_list = array(1,2,3);

// Atomic addition of a value to user_list
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```

#   Generic
*   `Collection<? extends Clazz>`
*   `Collection<? super Clazz>`
##  when?
*   use an `extends` wildcard when you only get values out of a structure
*   use a `super` wildcard when you only put values into a structure
*   Don't use a wildcard when you both want to get and put from/to a structure
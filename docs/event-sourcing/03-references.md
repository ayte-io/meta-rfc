---
title: References
---

The first issue in simplest example is storing type FQCNs, which can 
lead to unpleasant consequences, including inability to relocate types
within an application or restoring events in several applications (this 
may be a bit more common than expected: a typical application would
have methods for both returning at particular version and list
of mutations, and UI will have to render mutations for end user).

Such problem is easily addressed with a level of abstraction. It is
recommended to store particular mutation names (`hold`, `freeze` and so
on) and keep name-to-type mapping in language runtime.

```
interface TypeMapping {
  Class<?> typeOf(String name);
  String nameOf(Class<?> type);
}
```

Such approach allows to do virtually anything to in-language types and
preserve normal operation as long as all names are present in mapping.

Same goes with entities: instead of persisting `org.company.Account` it
is recommended to persist just `account`.

The last thing to mention is a very strange edge case that this pattern
also solves. Even though it is perceived as complete absurd on first 
read, it is possible that single class may be mapped to several 
entities. Classic example of such is a simple string, going hyperbolical
it is possible that application will manage texts as strings (with 
mutations like substitutions, insertions and so on) and usernames as
strings (so user with id = {uuid} would change his username from 
`@petya-21-years-old` to `@petya-22-years-old` on his birthday). In such
case it is not possible to use classes for direct entity reference 
without additional abstraction layer, i.e. class-to-name type mapping.

# LINQ Dynamic

## Description
You can execute query dynamically through free extensions provided by the library [Eval-Expression.NET](http://eval-expression.net/).

This feature is available for free in the EF Classic Community.

### Predicate

All LINQ predicate methods are supported. A string expression which return a Boolean function can be used as parameter.

###### Deferred Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `Where` | | [Try it](https://dotnetfiddle.net/QhVfRW) |

###### Immediate Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `All` | | [Try it](https://dotnetfiddle.net/YCT73M) |
| `Any` | | [Try it](https://dotnetfiddle.net/vEbwLr) |
| `Count` | | [Try it](https://dotnetfiddle.net/v8rqKV) |
| `First` | | [Try it](https://dotnetfiddle.net/CfxUKL) |
| `FirstOrDefault` | | [Try it](https://dotnetfiddle.net/UX3Ymb) |
| `LongCount` | | [Try it](https://dotnetfiddle.net/4xrM1d) |
| `Single` | | [Try it](https://dotnetfiddle.net/onW4hW) |
| `SingleOrDefault` | | [Try it](https://dotnetfiddle.net/nU97uw) |

```csharp
var list = context.Customers.Where(x => "x.IsActive").ToList();
var list2 = context.Customers.Where(x => "x.IsActive == IsActive", new { IsActive = false }).ToList();
```
[Try it](https://dotnetfiddle.net/GTttpq)

## Order & Select

All LINQ selector and order are supported. Most of them require the "Dynamic" suffix to not override default behavior (Ordering or selecting by a string is valid).

###### Deferred  Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `OrderByDescendingDynamic` | | [Try it](#) |
| `OrderByDynamic` | | [Try it](#) |
| `SelectDynamic` | | [Try it](#) |
| `SelectManyDynamic` | | [Try it](#) |
| `ThenByDescendingDynamic` | | [Try it](#) |
| `ThenByDynamic` | | [Try it](#) |

```csharp
var list = context.Customers.OrderByDescendingDynamic(x => "x.Name").ToList();
```
[Try it](https://dotnetfiddle.net/Fwjgin)

## Execute

The Execute method is the LINQ Dynamic ultimate methods which let you evaluate and execute a dynamic expression and return the result.

###### Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `Execute` | | [Try it](#) |
| `Execute<TResult>` | | [Try it](#) |

```csharp
var list = context.Customers.Execute<IEnumerable<Customer>>("Where(x => x.IsActive == true)").ToList();
var list2 = context.Customers.Execute<IEnumerable<Customer>>("Where(x => x.IsActive == IsActive)", new { IsActive = false }).ToList();
```
[Try it](https://dotnetfiddle.net/7S3JS0)


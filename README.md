# elm-static-array

Use Arrays with fixed length! All checks are done in compile time. Is this possible? Yes it is!
The magic word is _Phantom types_.

The package contains three types:

* `StaticArray a` - A wrapper type over `(a,Array a)` representing a non-empty array. Length is known in compile time.
* `Index n` - A wrapper type over `Int`. Allows values between `0` and `n`. It is essentially a `Range` type. Checks are done in run time but the max-value is known in compile time.
* `Length n` - A wrapper type over `Int`. Values can not be changed in run time. The value is known in compile type, thus making the magic happen.

**Pros**

* Length is checked in compile time by using phantom types.
* Wrapper Types ensure that no performance is lost.
* Convert Custom Types into `Int` and back without needing a dead branch in the case distinction.

**Cons**

* Construction is a bit slower (should be neglectable for most cases).
* If you don't know why this package is useful, you should not be using it.


# Examples

## Construction

```
StaticArray.fromList (Length.five |> Length.plus1) [0,1,2,3,4,5]
```

## Appending Elements

```
six =
  Length.five |> Length.plus1

twelve =
  Length.ten |> Length.plus2

array1 =
  StaticArray.fromList six [0,1,2,3,4,5]
  |> StaticArray.toRecord

array2 =
  StaticArray.fromList six [0,1,2,3,4,5]
  |> StaticArray.toRecord

StaticArray.fromRecord
  { length = twelve
  , head = array1.head
  , tail = Array.append (array1.tail |> Array.push array2.head) array2.tail
  }
```

Notice that we can NOT do addition in compile time, therefore we need to construct 2*6 manually.

## Removing Elements

```
six =
  Length.five |> Length.plus1

array =
  StaticArray.fromList six [0,1,2,3,4,5]

array
  |> StaticArray.resize (six |> Length.minus1)
  --> StaticArray.fromList Legnth.five [0,1,2,3,4]
```

## Avoiding index out of bounds errors

```
array =
  StaticArray.fromList Length.five [0,1,2,3,4]

fifthIndex =
  array |> Index.last (Array |> StaticArray.legnth)

array
  |> StaticArray.resize Lenght.one
  |> StaticArray.get fifthIndex --COMPILER ERROR
```

## Converting into/from Int

```
type Food =
  Apples
  | Oranges

asArray : StaticArray Two Food
asArray =
  StaticArray.fromList Length.two [Apples,Oranges]

toInt : Food -> Int
toInt food =
  case food of
    Apples -> 0
    Oranges -> 1

fromInt : Int -> Food
fromInt int =
  asArray
    |> StaticArray.get (int |> Index.fromModBy Length.two)
```

No dead branch needed.  
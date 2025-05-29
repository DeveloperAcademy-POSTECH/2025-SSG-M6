>[!question]
>GQ1. 왜 COW가 필요한 것일까? (COW가 있음으로써 어떠한 장점이 있는거지?)
>GQ2. 얕은 복사 (Shallow Copy)는 뭐고 깊은 복사 (Deep Copy)는 뭐고?

## Description

- COW (Copy-On-Write)는 값이 여러 곳에서 공유될 때, **수정이 발생할 때만 복사를 수행하는 전략**이다.
- 처음에는 얕은 복사 (Shallow Copy)가 수행 -> 수정 (Write)하려 할 때, 원본이 다른 곳에서 공유 중이라면 그때 깊은 복사 (Deep Copy)가 발생한다는 원리
- Swift 표준 라이브러리의 [[컬렉션 타입 (Collection Type)]]는 기본적으로 COW를 적용하고 있음.

##### 잠깐! 얕은 복사 (Shallow Copy)는 뭐고, 깊은 복사는 뭔데?

> 이 둘은 메모리에서 Swift의 데이터를 어떻게 복사하는가, 그리고 복사된 데이터가 원본 데이터와 얼마나 독립적인가에 대한 이야기이다.

- 얕은 복사 (Shallow Copy) : 객체의 참조만 복사, 내부 데이터는 사실 공유되고 있음 -> [[참조 타입 (Reference Type)]]의 복사 전략
- 깊은 복사 (Deep Copy) : 객체를 아예 새로 만들어서, 내부 데이터의 내용물까지 모두 복사하는 것 -> [[값 타입 (Value Type)]]의 복사 전략
- 만약, 참조 타입에서도 깊은 복사 (Deep Copy)를 적용하고 싶다면, 직접 clone 메서드를 구현하거나 / Objective-C 스타일의 NSCopying을 사용할 수 있다.


## Why COW? | 왜 Copy-On-Write가 필요한거지?

+ 너무나 당연하게도 성능이 좋아지기 때문이다.
+ 조금 더 자세하게 들어가자면...

	+ 최대한 필요한 경우 (= Write가 이루어지는 상황)에만 복사가 이루어지니, 메모리 낭비를 최소화할 수 있다.
	+ 수정 시에 비로소 깊은 복사가 이루어지니, 원본 데이터를 최대한 보호한 상황에서 복사가 이루어질 수 있다.
	
- 즉, 얕은 복사의 빠른 속도 + 깊은 복사의 안정성을 모두 결합하는 최적의 방법이 COW인 것.



## 코드 예시

+ a와 b는 Collection Type인 Array이므로, 얕은 복사에서 -> 깊은 복사로 바뀌는 상황을 확인할 수 있다.

```swift
var a = [1, 2, 3]
var b = a  // 참조만 복사됨 (깊은 복사 같지만, 사실 이 시점에서는 "얕은 복사")

print(a) // [1, 2, 3]
print(b) // [1, 2, 3]

b.append(4) // 이 시점 (Write가 발생한 시점)에 비로소 "깊은 복사"가 이루어진 것!

print(a) // [1, 2, 3]
print(b) // [1, 2, 3, 4]
```

- Swift에서는 COW 여부를 판단하기 위해 `isKnownUniquelyReferenced(_:)` 메서드를 사용할 수 있다.

```swift
final class Buffer {
    var storage: [Int]

    init(_ storage: [Int]) {
        self.storage = storage
    }

    func copy() -> Buffer {
        return Buffer(self.storage)
    }
}

struct MyArray {
    private var buffer: Buffer

    init(_ storage: [Int]) {
        self.buffer = Buffer(storage)
    }

    var elements: [Int] {
        return buffer.storage
    }

    mutating func append(_ element: Int) {
        // 여기서 공유 여부를 판단 (파라미터 타입이 final class 타입에서만 의미있다!)
        if !isKnownUniquelyReferenced(&buffer) {
            buffer = buffer.copy() // 깊은 복사가 이 시점에서 발생!
        }
        buffer.storage.append(element)
    }
}
```


## Keywords

- [[컬렉션 타입 (Collection Type)]]
- [[참조 타입 (Reference Type)]]
- [[값 타입 (Value Type)]]


## References

- [Swift에서 noncopyable 유형 소비하기](https://developer.apple.com/kr/videos/play/wwdc2024/10170/)
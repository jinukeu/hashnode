---
title: "단위테스트 2장 정리"
datePublished: Tue Sep 03 2024 12:38:26 GMT+0000 (Coordinated Universal Time)
cuid: cm0mf0zqj000509ihc2866rjq
slug: 2

---

단위 테스트의 정의에는 많은 늬앙스가 있다. 크게 고전파, 런던파로 나뉜다.

고전파: 모든 사람이 단위 테스트와 테스트 주도 개발에 원론적으로 접근하는 방식

런던파: 런던의 프로그래밍 커뮤니티에서 시작

# 단위 테스트의 정의

단위 테스트는

* 작은 코드 조각을 검증하고
    
* 빠르게 수행하고
    
* 격리된 방식으로 처리하는 자동화된 테스트다.
    

## 격리 문제에 대한 런던파의 접근

코드 조각을 격리된 방식으로 검증한다는 것은 무엇을 의미할까?

런던파에서는 테스트 대상 시스템을 협력자에게서 격리하는 것을 일컫는다. 즉, 하나의 클래스가 다른 클래스 또는 여러 클래스에 의존하면 이 모든 의존성을 테스트 대역(영화의 스턴트 대역과 비슷한 의미)로 대체해야 한다.

장점)

1. 테스트가 실패하면 코드베이스의 어느 부분이 고장 났는지 알 수 있다.
    
2. 클래스간의 의존성을 분할할 수 있다.
    

고전파 예시)

```csharp
public void Purchase_succeeds_when_enough_inventory()
{
  var store = new Store();
  store.AddInventory(Product.Shampoo, 10);
  
  bool success = customer.Purchase(store, Product.Shampoo, 5);
  
  Assert.True(success);
  Assert.Equal(5, store.GetInventory(Product.Shampoo));
}
```

테스트 대상 시스템 : 고객

협력자 : 상점

Customer에 영향을 미치는 Store 내부에 버그가 있으면 단위 테스트에 실패할 수 있다. 테스트에서 두 클래스는 서로 격리되어 있지 않다.

런던파 예시)

```csharp
public void Purchase_succeeds_when_enough_inventroy()
{
  var storeMock = new Mock<IStore>();
  storeMock
    .setUp(x => x.HasEnoughInventroy(Product.Shampoo, 5))
    .Returns(true);
  var customer = new Customer();
  
  bool success = customer.Purchase(
      storeMock.Object, Product.Shampoo, 5);
      
  Assert.True(success);
  storeMock.Verify(
      x = x.RemoveInventory(Product.Shampoo, 5),
      Times.Once);
 }

```

실제 인스턴스가 아닌 목을 사용한다.

고전파에서는 상점의 상태를 검증했다.

런던파에서는 상호 작용을 검사한다. (고객이 상점에서 호출을 올바르게 했는가? 고객이 상점으로 호출해야 하는 메서드 뿐만 아니라 횟수까지 검증)

## 격리 문제에 대한 고전파의 접근

각각의 테스트를 격리하는 것은 여러 클래스가 모두 메모리에 상주하고 공유 상태에 도달하지 않는 한, 여러 클래스를 한번에 테스트해도 괜찮다. (예를 들면, 데이터베이스, 파일 시스템, static 필드)

A 테스트에서 데이터베이스에 고객을 추가함. 추가된 고객이 B 테스트의 결과에 영향을 미치면 안됨

# 단위 테스트의 런던파와 고전파

|  | 격리 주체 | 단위의 크기 | 테스트 대역 사용 대상 |
| --- | --- | --- | --- |
| 런던파 | 단위 | 단일 클래스 | 불변 의존성 외 모든 의존성 |
| 고전파 | 단위 테스트 | 단일 클래스 또는 클래스 세트 | 공유 의존성 |

## 고전파와 런던파가 의존성을 다루는 방법

런던파에서 불변 객체는 테스트 대역으로 교체하지 않아도 된다.

아래 예시에서 Product는 테스트 대역을 사용하지 않는다.

```csharp
public void Purchase_fails_when_not_enough_inventory()
{ 
 var storeMock = new Mock<IStore>();
 storeMock
  .Setup(x => x.HasEnoughInventory(Product.Shampoo, 5))
  .Returns(false);
  
  bool success = customer.Purchase(storeMock.Object, Product.Shampoo, 5);
  
  // ... 생략
```

# 고전파와 런던파의 비교

## 런던파

### 장점

* 입자성이 좋다. 테스트가 세밀해서 한 번에 한 클래스만 확인한다
    
* 서로 연결된 클래스의 그래프가 커져도 테스트하기 쉽다. (테스트 대역을 사용하기 때문)
    
* 테스트가 실패하면 어떤 기능이 실패했는지 확실히 알 수 있다.
    

### 단점

* 응집도가 낮고 일반 사람들에게 의미가 없다.
    
    * 고전파 스타일 예시: 우리집 강아지를 부르면, 바로 나에게 온다
        
    * 런던파 스타일 예시: 우리집 강아지를 부르면 먼저 왼쪽 앞다리를 움직이고, 이어서 오른쪽 앞다리를 움직이고 …
        
        * 고전파 스타일이 더 의미있다.
            
* 애초에 서로 연결된 클래스의 그래프가 커지면 안된다.
    
* 런던파 테스트 시스템에 버그가 생기면 SUT에 버그가 포함된 테스트만 실패한다. 고전파의 경우 오작동하는 클래스를 참조하는 클라이언트에도 버그가 발생할 수 있다. 고전파가 더 가치있다.
    

# 두 분파의 통합 테스트

런던파: 실제 협력자 객체를 사용하는 모든 테스트를 통합 테스트로 간주한다

고전파: 공유 의존성, 프로세스 외부 의존성뿐 아니라 조직 내 다른 팀이 개발한 코드 등과 통합해 작동하는지도 검증하는 테스트

### 엔드 투 엔드 테스트

최종 사용자 관점에서 시스템을 검증한다.
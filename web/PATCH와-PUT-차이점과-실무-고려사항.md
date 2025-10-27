작성일 | 251016

# 배경

PATCH는 PUT에 비해 언급이 적고 공식문서에서 많이 다뤄지지 않는다. 

심지어는 PATCH를 실무에서 잘 안다룬다는 블로그 글도 있어서 왜 그렇게 이야기하는지 한번 정리해봤다. 

# 분석

다른 블로그 글을 분석하기에 앞서 내가 알고 있는 PATCH와 PUT의 차이점에 대해서 짚고 가보자면 

PATCH : 리소스에 대한 수정을 요청하면 부분에 대해서만 수정이 일어난다. 다른 필드의 값은 보존한다. 

PUT: 리소스에 대한 수정을 요청하면 전체를 교체해버린다. 다른 필드의 값은 무시한다. 

## PUT과 PATCH는 관점의 차이를 갖는다.
    
```java
@Entity
class Article {
    @Id @GeneratedValue Long id;
    String title, content;
    long viewCount = 0;

    void incrementViewCount() { viewCount++; }
    void update(ArticleUpdateRequest req) {
        title = req.title;
        content = req.content;
    }
}
```

사용자 요청 

```java
{
    "title": "수정할 글 제목",
    "content": "수정할 글 내용"
}
```

이렇게 들어올 때, 

- 서버는 “title과 content만 변경하는 군!” 하고 PATCH를,
- 클라이언트는 “게시글의 제목과 내용을 바꾸니 모두 변경하는군!” 하고 PUT 을 생각한다.

**여기서 서버의 관점을 적용해서 PATCH를 사용하는 경우**

1. 클라이언트는 수정할 부분에 대해서만 요청을 보낸다. 
2. 위의 update 함수에서는 수정할 부분이 아닌 경우는 null로 대체한다.
3. 그러면 클라이언트는 title(content)만 수정하려고 했는데 content(title)은 null이 되어 사라짐

그래서 update 함수를 수정해보자. 

update 함수를 수정한다면, 

```java
@Entity
class Article {
    @Id @GeneratedValue Long id;
    String title, content;
    long viewCount = 0;

    void incrementViewCount() { viewCount++; }
    void update(ArticleUpdateRequest req) {
            if(!req.title.equals(null)){
                title = req.title
                }
            if(!req.content.equals(null)){
                content = req.content
                }
    }
}
```

이렇게 null 값 검사를 하는 방식으로 수정할 수 있다. 

여기서 만약 제목을 바꾸고 싶지 않다면 어떻게 할까? 이 경우에도 두 가지 요청 예시가 존재한다. 서로 다른 의도를 갖지만 처리 결과는 동일하다. 

```java
{
    "title": null,
    "content": "수정할 글 내용"
}
```

```java
{
    "content": "수정할 글 내용"
}
```

이런 요청을 해결하기 위해서 Map을 사용할 수도 있다. 

```java
@Entity
class Article {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String title, content;
    long viewCount = 0;

    protected Article() {}

    void increaseViewCount() { viewCount++; }

    void update(Map<String, Object> req) {
        if (req.containsKey("title")) title = (String) req.get("title");
        if (req.containsKey("content")) content = (String) req.get("content");
    }
}
```

이 방식은 Dto의 장점을 포기해야 한다. 

- Dto의 장점이 뭐지?
    
    데이터 흐름을 추상적으로 보여줄 수 있는 것  + 어떤 데이터를 다루는지 쉽게 알 수 있음 
    
    그런데 Map으로 하면 데이터가 숨겨진다. → 데이터에 접근하기 위한 단계가 추가된다?
    

이 외에도 치명적인 부분은 Dto 빈으로 어노테이션을 통한 유효성 검사를 진행할 수 없다는 점에도 있을 것 같다. 

**이것 말고도 도메인 속성이 변경되는 상황을 고려해보자.** 

```java
class User{
    Long id; 
} 
```

```java
class User{
    Long id; 
    String name; 
    String phoneNum;
} 
```

변경 전 id를 수정하는 메서드를 PUT으로 개발했다. (필드는 id밖에 없으므로) 

여기서 도메인 속성이 변경되면 이 함수는 어떻게 수정해야 할까요? 수정하는 방식은 두 가지가 있다. 

그럼에도 두 방식 모두 클라이언트에서 요청하는 내용에 대한 수정이 필요하다.  

1. (PATCH로 변경) Id 필드만 수정하는 메서드
    1. 클라이언트는 계속 PUT으로 사용하고 있어 오류가 난다. 
2. (PUT 유지) 모든 필드를 수정하는 메서드
    1. 클라이언트가 이제 전체 필드에 대한 데이터를 보내야만한다. (아니면 데이터 전부 유실)

### 정리하자면 
- PATCH를 사용하게 되면 PATCH 동작을 위해서 추가하고 처리해줘야하는 조건이 늘어난다. 
    - PATCH 동작: 필드에 대한 요청이 들어오면 다른 필드는 유지하고 해당 필드만 수정된다. 
    - PUT에 비하면 복잡한 조건이 존재한다. 


- api 사용 스펙을 바꾸고 수정하는 일은 서버에서만 수정하는게 아니라 클라이언트의 수정도 필요하다. 그래서 우리는 클라이언트의 관점에서 생각할 수 있어야 한다. 

- 클라이언트의 관점에 맞는 메서드는 PUT이다. 
    - 클라이언트는 서버 내부 동작과 무관하게 요청하는대로 데이터가 교체되고 수정되기를 원하기 때문에 

## PATCH를 사용할거면 어떻게 써야 할까?
서로 다른 4개의 필드를 수정하는 함수를 설계한다. 이때 고려할 수 있는 방식은 두 가지이다. 
    
1. **모든 필드의 값을 받는 함수**
- 변경되지 않은 값을 포함해 모든 필드의 값을 받는다.
- PUT스럽다.
    
2. **일부 필드의 값을 받는 함수** 
- 변경되는 값만 받고 그에 대한 수정만 진행한다.
- PATCH 스럽다.
    
1번 함수의 경우 → 요청받은 변경 값 + 요청받지 않은 기존 값을 모두 수집해야 한다. 
    
2번 함수의 경우 → 변경되는 값과(요청값), 변경되지 않는 null 값으로 나누어 생각할 수 있다. 그래서 수정 시 null 체크가 필요하다. 
    
여기에 더해서 null이 의미하는 바에 대해서도 한번 명확하게 짚어봐야 한다. 
    
null은 (1) null로 수정하겠다. (2) 기존 값을 유지하겠다. 의 두 가지 의미로 다르게 사용될 수 있다. 
    
## PATCH는 이미 존재하는 데이터에 대한 수정을 의미한다.
    
PUT은 새로운 리소스를 생성하거나, 기존 리소스를 교체하는 역할을 수행하기도 한다.
좋아요 기능, 찜 기능이 PUT메서드를 활용하기에 적합하다고 이야기한다. PUT으로 존재하지 않는 찜을 만들거나 기존 찜의 상태를 바꿀 수 있기 때문이다. 
    
PATCH는 부분 수정이라는 특징에 맞게 이미 존재하는 엔티티에 대해 요청한 필드에 대한 수정을 진행한다. 
    

# 결과

**PUT과 PATCH의 차이**

- PATCH는 서버의 관점에 가깝다. 엔티티 필드 전체를 관리하기에 대부분의 수정 요청은 부분 수정이 된다. 이전에 존재하는 리소스에만 적용된다.
- PUT은 클라이언트의 관점에 더 가깝다. 클라이언트는 서버 내부 동작같은거 모르고 관심이 없다. 그냥 내가 보내는 대로 수정되면 그만이다. 리소스 생성에도 활용될 수 있다.

**PATCH를 사용할 때 주의할 점 : null**

PATCH 요청에서 주어지는 필드는 수정하고자 하는 필드 뿐이다. 서버의 입장에서 다른 필드에 대한 수정이 발생하는지 아닌지 확인하기 위해 null 값을 사용한다. 

수정 요청에 필드에 대한 요청이 null이 들어오는 경우 

수정 요청에 필드에 대한 요청이 없는 경우 

위의 두 가지 경우 모두 코드 상에서 null로 취급할 수 있다. null이 갖는 의미가 최소 두 가지가 된다는 건데, 그러면 예상치 못한 결과를 낳을 수 있다. 

그래서 PATCH를 사용할 때 null 체크가 중요하다. 조건 분기로 필드가 null인지 아닌지 확인하며 값을 다뤄야 한다. 

**실무에서 어떤 목적에서 사용될까?** 

1. 기능상 명확하게 부분 수정이 필요한 경우 
    
    사용자 프로필 변경 등 잦은 빈도로 부분 수정을 하게 된다면 굳이 다른 필드의 값을 다 보내는 것보다 수정 부분에 대해서만 요청을 보내는 것이 더 직관적이다. 
    
2. 데이터 필드가 복잡하거나 리소스 용량이 큰 경우 
    
    PUT을 사용하는 수정은 필드 전체에 대한 요청을 필요로 한다. 이때 매번 전체 데이터를 다 보내는 것은 비효율적이다.


**참고**

https://colabear754.tistory.com/233

https://yeonyeon.tistory.com/183

https://afuew.tistory.com/17

https://developer.mozilla.org/ko/docs/Web/HTTP/Reference/Methods/PUT

https://developer.mozilla.org/ko/docs/Web/HTTP/Reference/Methods/PATCH

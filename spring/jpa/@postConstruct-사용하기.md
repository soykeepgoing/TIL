작성일 | 251030 

# 문제 상황 

1. `Repository` 객체에 대해서 코드 리뷰 피드백을 받고 코드를 수정하는데 로그 찍어봐도 수정 사항이 안보였다. 
2. 알고 보니 빈 초기화 자체가 안되는 거 였다. 

# 코드 살펴보기 

문제 코드를 살펴보면 

```java
//Service.java
@Service("postLikeService")
public class PostLikeService extends LikeService {
    @Autowired
    public PostLikeService(UserCsvRepository userCsvRepository,
                           @Qualifier("postLikeRepository") LikeCsvRepository likeCsvRepository,
                           PostCsvRepository postCsvRepository) {
        ...
    }
}

//Repository.java

@Slf4j
@Repository("postLikeRepository")
public class PostLikeCsvRepository extends LikeCsvRepository{
    @Override
    public void init(){
        this.dbPath = Contents.POST.getDbPath();
        this.contentType = Contents.POST.getContentType();
        initFromFile();
    }
}

```

ocp 원칙을 적용하려고 이리 저리 숨겨놧더니 빈 생성 후 초기화하는 단계를 놓쳤다. 의존성 주입 후 사용 직전에 후처리하는 단계가 빠졌다.
그래서 어노테이션 `@PostConstruct`를 추가해 문제를 해결했다. 

`@PostConstruct`라는 것은 스프링 빈 생명 주기를 봤을 떄 

스프링 빈 생성 -> 의존성 주입 -> 후처리 (PostConstruct) [초기화 콜백 메서브 호출] -> 사용 -> 후처리 (PostConstruct) [소멸 콜백 메서드 호출] -> 소멸 

단계를 거쳐서 빈을 사용하는 것이기 때문에 이런 후처리 관련 어노테이션을 사용해야 한다. 

여기서 내가 의도했던 것은 likeRepository라는 레포지토리 빈을 생성하고 나서 해당 레포지토리에 대해 특정 경로에 대한 초기화 즉 후처리를 하길 원햇기 때문에 이렇게 문제를 해결할 수 있다. 
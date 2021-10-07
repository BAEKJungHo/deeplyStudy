# nextval 과 currval 의 차이점 및 Mybatis selectKey 에 대한 고찰

[OKKY ORACLE에서 시퀀스.currval 질문](https://okky.kr/article/348698)에 있는 예제를 살짝 바꿨다.

전제조건은 다음과 같다. 게시판에서 글쓰기와 첨부파일 등록을 하나의 서비스 메서드에서 처리중이고 트랜잭션이 걸려있는 상태이며, Mybatis 를 사용하고 있다.

이 예제에서 다음과 같은 고민을 할 수 있다.

- currval 을 쓰는게 맞는지, selectKey 를 쓰는게 맞는지에 대한 고민을 할 수 있다.


## References

- https://okky.kr/article/348698
- http://www.gurubee.net/article/61086
- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bluekisunny&logNo=120045145287
- https://developer-alle.tistory.com/203

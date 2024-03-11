## coupon.repository
```
@Injectable()
export class CouponRepository extends Repository<Coupon> {
  constructor(
    @InjectRepository(Coupon)
    private readonly repo: Repository<Coupon>,
    @InjectEntityManager()
    private readonly entityManager: EntityManager,
  ) {
    super(repo.target, repo.manager, repo.queryRunner);
  }
```
1. 쿠폰레포지토리 클래스는 레포<쿠폰> 을 상속받음

2. 클래스 생성자를 쿠폰Entity 레포를 주입하여, 레포<쿠폰> 의 기능을 사용할 수 있음

3. 앤티티메니저를 사용해 CRUD 작업을 하고 필드를 선언한다.

4. sueper 로 생성자를 호출하여 초기화 시킴
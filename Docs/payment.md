## Order Entity

1. ManyToOne user 한 사용자는 여러 주문을 할수 있음.

2. UserEntity와의 관계를 지정, 관계를 맺는 필드.

3. orderNo, amount, pointAmoutUserd 컬럼지정

4. status: export type OrderStatus = 'started' | 'paid' | 'refunded'; 중 하나의 타입을 가지고 있어야 함.

5. item은 일대다 관계이며, OrderItem 관계를 맺음

6. JoinColumn 일대일 관계에서 외래키를 나타냄

7. shippingInfo 배송정보 필드 mull값 허용

8. IssuedCoupon 일대일, null true

9. refundReason, refundedAmount, refundedAt 환불사유, 환불 금액, 환불된 시간 정의

```
  constructor() {
    super();
    this.setOrderNo();
  }

  setOrderNo() {
    const date = new Date();
    const dateFormat = `${date.getFullYear()}${String(
      date.getMonth() + 1,
    ).padStart(2, '0')}${String(date.getDate()).padStart(2, '0')}${String(
      date.getHours(),
    ).padStart(2, '0')}${String(date.getMinutes()).padStart(2, '0')}${String(
      date.getSeconds(),
    ).padStart(2, '0')}`;
    const randomString = Array.from(
      { length: 15 },
      () => Math.random().toString(36)[2] || '0',
    ).join('');
    this.orderNo = `${dateFormat}_${randomString.toUpperCase()}`;
  }
```

10. setOrderNo 생성 날짜, 시간 기반하여 주문번호 생성 로직

## CreateOrderDto

```
export type CreateOrderDto = {
  userId: string;
  orderItems: OrderItem[];
  couponId?: string;
  pointAmountToUse?: number;
  shippingAddress?: string;
};
```

1. 선택적인 사항 쿠폰, 사용할 포인트, 배송 주소 지정을 선택할 수 있는로직

## Coupon Entity
```
type CouponType = 'percent' | 'fixed';

@Entity()
export class Coupon extends BaseEntity {
  @Column({ type: 'varchar', length: 50 })
  type: CouponType;

  @Column({ type: 'decimal', precision: 5, scale: 2 })
  value: number; // 할인율 또는 정액 금액

  @OneToMany(() => IssuedCoupon, (issuedCoupon) => issuedCoupon.coupon)
  issuedCoupons: Relation<IssuedCoupon[]>;
```

1. couponTupe 을 percent(할인), fixed(정액할인) 로 종류를 나타냄

2. BaseEntity를 상속받으며 CouponType에 일치하는 값을 나타나며, 쿠폰은 varcahr 타입에 길이는 50 제한을 둠

3. decimal 타입의 총자릿수, scale은 소수점 이하 자릿수 나타냄 할인율, 정액 금액을 나타냄

4. 하나의 쿠폰은 발급된 여러 쿠폰을 가질수 있는 oneToMany 설정 IssuedCoupon Entity와 관계지정

## IssudeCoupon Entity
```
@Entity()
export class IssuedCoupon extends BaseEntity {
  @ManyToOne(() => User)
  @JoinColumn()
  user: Relation<User>;

  @ManyToOne(() => Coupon)
  @JoinColumn()
  coupon: Relation<Coupon>;

  @OneToOne(() => Order, (order) => order.usedIssuedCoupon, { nullable: true })
  usedOrder: Relation<Order>;

  @Column({ type: 'boolean', default: false })
  isValid: boolean;

  @Column({ type: 'timestamp', nullable: false })
  validFrom: Date;

  @Column({ type: 'timestamp', nullable: false })
  validUntil: Date;

  @Column({ type: 'boolean', default: false })
  isUsed: boolean;

  @Column({ type: 'timestamp', nullable: true })
  usedAt: Date;
```

1. user entity와 다대일 관계이며, JoinCoulme 이기에 다대일 관계에서 외래키를 나타내고, 다대일 관계이므로 단일 사용자를 참조

2. coupon 관계는 다대일 이며, 하나의 발금된 쿠폰을 나타냄

3. userOrder 와 order 관계가 일대일 관계를 나타내고 쿠폰은 하나의 주문에 사용가능

4. isValid: boolean 타입, 쿠폰읜 유효성

   validFrom: Date 타입 쿠폰유효 시작일

   isUsed: Date 타입, 쿠폰 종료 일자

5.

```
  use() {
    this.isUsed = true;
    this.isValid = false;
    this.usedAt = new Date();
  }
```

isUsed 사용여부, usedAt 사용일자, use 쿠폰의 상태를 변경

## OrderItem Entity
```
export type OrderStatus = 'started' | 'paid' | 'refunded';

@Entity()
export class Order extends BaseEntity {
  @ManyToOne(() => User, (user) => user.orders)
  user: User;

  @Column({ type: 'varchar' })
  orderNo: string;

  @Column()
  amount: number;

  @Column({ type: 'varchar', length: 100 })
  status: OrderStatus;

  @OneToMany(() => OrderItem, (item) => item.order)
  items: Relation<OrderItem[]>;

  @Column({ type: 'int', default: 0 })
  pointAmountUsed: number;

  @OneToOne(() => IssuedCoupon, (issuedCoupon) => issuedCoupon.usedOrder, {
    nullable: true,
  })
  @JoinColumn()
  usedIssuedCoupon: Relation<IssuedCoupon>;

  @OneToOne(() => ShippingInfo, (shippingInfo) => shippingInfo.order, {
    nullable: true,
  })
  @JoinColumn()
  shippingInfo: Relation<ShippingInfo>;

  @Column({ type: 'text', nullable: true })
  refundReason: string;

  @Column({ type: 'decimal', nullable: true })
  refundedAmount: number;

  @Column({ type: 'timestamp', nullable: true })
  refundedAt: Date;

  @Column({ type: 'jsonb', nullable: true })
  pgMetadata: any; // PG사 메타데이터

  constructor() {
    super();
    this.setOrderNo();
  }

  setOrderNo() {
    const date = new Date();
    const dateFormat = `${date.getFullYear()}${String(
      date.getMonth() + 1,
    ).padStart(2, '0')}${String(date.getDate()).padStart(2, '0')}${String(
      date.getHours(),
    ).padStart(2, '0')}${String(date.getMinutes()).padStart(2, '0')}${String(
      date.getSeconds(),
    ).padStart(2, '0')}`;
    const randomString = Array.from(
      { length: 15 },
      () => Math.random().toString(36)[2] || '0',
    ).join('');
    this.orderNo = `${dateFormat}_${randomString.toUpperCase()}`;
  }
```

1. 다대일 관계 하나의 주문 항목은 order 에 속하며 양방향관계임을 나타내기 위해 order.items 로 역관계를 정의 Relation<Order> 다대일 관계 이므로 단일 주문을 참조

2. productId, Quantity 로 주문Id, 수량 정의

## pointLog Entity

```
export type PointLogType = 'earn' | 'spend';

@Entity()
export class PointLog extends BaseEntity {
  @ManyToOne(() => Point, (point) => point.logs)
  point: Relation<Point>;

  @Column({ type: 'int', default: 0 })
  amount: number; // 개별 적립 또는 사용 금액

  @Column({ type: 'text' })
  reason: string; // 개별 적립 또는 사용 사유

  @Column({ type: 'varchar', length: 50 })
  type: PointLogType;

  use(amount: number, reason: string): void {
    this.amount = amount;
    this.reason = reason;
    this.type = 'spend';
  }

  add(amount: number, reason: string): void {
    this.amount = amount;
    this.reason = reason;
    this.type = 'earn';
  }
```

1. PointLogType 은 earn, spend 타입중 하나를 가질수 있다.
   (earn: 포인트 획득 로그, spend: 포인트를 사용한 로그)

2. pontLog 클래스는 BaseEntity 클래스를 상속받음

3. 다대일 관계로 하나의 포인트에 하나의 포인트 로그가 속하며, 양방향 관계임을 나타내기 위해 point.logs로 역 관계를 정의

4. point: Relation<Point> 다대일 관계이므로 단일 포인트를 참조

5. amount: number 는 포인트 적립 또는 사용 금액을 나타냄, reason: string 사용 사유? 를 나타냄, type: PointLogType 포인트 로그의 종류를 나타내는 필드 'earn' 또는 'spend' 중 하나의 값을 가진다.

6. use(amount: number, reason: string): void 주어진 금액과 이유를 설정하고 로그 타입을 'spend'로 변경

7. add(amount: number, reason: string): void 주어진 금액과 이유를 설정하고 로그 타입을 'earn'로 변경

## product Entity

1. available, out-of-stock 둘중 하나의 타입을 가질수 있고 재고의 상태를 나타냄

2. name, price, stock, category, imageUrl, description, status 각각의 컬럼을 가지고 있다.

## shippinginfo Entity
```
@Entity()
export class ShippingInfo extends BaseEntity {
  @OneToOne(() => Order, (order) => order.shippingInfo)
  order: Relation<Order>;

  @Column({ type: 'text' })
  address: string;

  @Column({ type: 'varchar', length: 50 })
  status: ShippingStatus;

  @Column({ type: 'varchar', length: 100, nullable: true })
  trackingNumber: string;

  @Column({ type: 'varchar', length: 50, nullable: true })
  shippingCompany: string;
}
```

1. type은 ordered: 주문완료, shipping: 출고 준비중, shipped: 출고 완료, delivering: 배송중, delivered: 배송 완료 타입을 가지고 있음.

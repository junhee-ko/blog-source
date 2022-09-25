---
layout: post
title: Hexagonal Architecture - Persistence Adapter
date: 2022-09-25
categories: Architecture
---

육각형 아키텍처에서 영속성 어뎁터를 구현해보자.

## 의존성 역전

다음은 영속성 어뎁터가 애플리케이션 서비스에 영속성 기능을 제공하기 위해, 의존성 역전 원칙을 적용했다.

![](/image/persistence-adapter-and-port.png)

애플리케이션 서비스에서 영속성 기능을 사용하기 위해, 포트 인터페이스를 호출한다. 이 포트는 영속성 작업을 수행하고 DB 와 통신할 책임을 가진 영속성 어뎁터 클래스에 의해 구현된다.

육각형 아키텍처에서의 영속성 어뎁터는 outgoing 어뎁터이다. 애플리케이션에 의해 호출된 뿐, 호출되지 않기 때문이다.

중요한 것은, 영속성 계층에 대한 의존성을 없애기  위해 간접 계층인 포트를 사용하고 있다는 것이다. 그래서, 영속성 코드를 수정하더라도 코어 코드를 변경하지 않을 수 있다.

## 영속성 어뎁터의 책임

1. 입력을 받는다. (포트 인터페이스를 통해)
2. 입력을 DB 포맷으로 매핑한다. (DB 를 쿼리하거나 변경하는데 사용할 수 있는 포맷으로)
3. 입력을 DB 에 보낸다.
4. DB 의 출력을 애플리케이션 포맷으로 매핑한다.
5. 출력을 반환한다.

## 포트 인터페이스 나누기

보통 특정 엔티티가 필요로 하는 모든 DB 연산을 하나의 repository interface 에 두는 게 일반적이다.
이렇게 되면,

1. DB 연산에 의존하는 각 서비스는 인터페이스의 메서드를 하나만 사용해도, 넓은 포트 인터페이스에 의존성을 가지게 된다.
2. 또한, 코드를 이해하기도 어렵고 테스트하기도 어려워진다. 

그래서, 인터페이스 분리 원칙을 적용해서 클라이언트 자신이 필요로 하는 메서드만 알 수 있도록 인터페이스를 특화된 인터페이스로 분리해야한다.

![](/image/port-interface-with-isp.png)

## 영속성 어뎁터 나누기

위 그림에서는, 모든 영속성 포트를 구현한 한 영속성 어뎁터 클래스가 있다. 하지만 아래와 같이 분리해서, 영속성 연산이 필요한 도메인 클래스 하나당 하나의 영속성 어뎁터를 구현하게 된다.

![](/image/divided-persistence-adapter.png)

## Spring Data JPA example

AccountPersistenceAdapter 를 구현한 코드를 보자. 이 어댑터는 DB 로부터 계좌를 가져오거나 저장할 수 있어야한다.

Account 엔티티를 보자.

```kotlin
package com.example.buckpal.domain  
  
data class Account(  
  val id: AccountId? = null,  
  val baselineBalance: Money,  
  val activityWindow: ActivityWindow  
) {  
  
  fun calculateBalance(): Money =  
    Money.add(  
      this.baselineBalance,  
      this.activityWindow.calculateBalance(id)  
    )  
  
  fun withdraw(money: Money, targetAccountId: AccountId): Boolean {  
    if (!mayWithdraw(money)) return false  
  
    val withdrawal = Activity(  
      ownerAccountId = id,  
      sourceAccountId = id,  
      targetAccountId = targetAccountId,  
      timestamp = LocalDateTime.now(),  
      money = money  
    )  
    activityWindow.addActivity(withdrawal)  
  
    return true  
  }  
  
  private fun mayWithdraw(money: Money): Boolean =  
    Money.add(  
      a = calculateBalance(),  
      b = money.negate()  
    ).isPositiveOrZero  
  
  fun deposit(money: Money, sourceAccountId: AccountId): Boolean {  
    val deposit = Activity(  
      id,  
      sourceAccountId,  
      id,  
      LocalDateTime.now(),  
      money  
    )  
    activityWindow.addActivity(deposit)  
  
    return true  
  }  
  
  companion object {  
  
    fun withoutId(  
      baselineBalance: Money,  
      activityWindow: ActivityWindow  
    ) = Account(  
      id = null,  
      baselineBalance = baselineBalance,  
      activityWindow = activityWindow  
    )  
  
    fun withId(  
      accountId: AccountId,  
      baselineBalance: Money,  
      activityWindow: ActivityWindow  
    ) = Account(  
      id = accountId,  
      baselineBalance = baselineBalance,  
      activityWindow = activityWindow  
    )  
  }  
  
  data class AccountId(  
    val value: Long  
  )  
}
```

DB 와의 통신에 JPA 를 사용할 것이다. @Entity 가 추가된 클래스가 필요하다.

```kotlin
package com.example.buckpal.adapter.out.persistence

@Entity  
class AccountJpaEntity {  
    
  @Id  
  @GeneratedValue  private val id: Long? = null  
}

@Entity  
class ActivityJpaEntity {  
  
  @Id  
  @GeneratedValue  private val id: Long? = null  
  
  @Column  
  private val timestamp: LocalDateTime? = null  
  
  @Column  
  private val ownerAccountId: Long? = null  
  
  @Column  
  private val sourceAccountId: Long? = null  
  
  @Column  
  private val targetAccountId: Long? = null  
  
  @Column  
  private val amount: Long? = null  
}
```

그리고, repository interface 를 추가하자.

```kotlin
interface AccountRepository : JpaRepository<AccountJpaEntity, Long>

interface ActivityRepository : JpaRepository<ActivityJpaEntity, Long> {  
  
  @Query(  
    "select a from ActivityJpaEntity a " +  
      "where a.ownerAccountId = :ownerAccountId " +  
      "and a.timestamp >= :since"  
  )  
  fun findByOwnerSince(  
    @Param("ownerAccountId") ownerAccountId: Long?,  
    @Param("since") since: LocalDateTime?  
  ): List<ActivityJpaEntity>  
  
  @Query(  
    ("select sum(a.amount) from ActivityJpaEntity a " +  
      "where a.targetAccountId = :accountId " +  
      "and a.ownerAccountId = :accountId " +  
      "and a.timestamp < :until")  
  )  
  fun getDepositBalanceUntil(  
    @Param("accountId") accountId: Long?,  
    @Param("until") until: LocalDateTime?  
  ): Long?  
  
  @Query(  
    ("select sum(a.amount) from ActivityJpaEntity a " +  
      "where a.sourceAccountId = :accountId " +  
      "and a.ownerAccountId = :accountId " +  
      "and a.timestamp < :until")  
  )  
  fun getWithdrawalBalanceUntil(  
    @Param("accountId") accountId: Long?,  
    @Param("until") until: LocalDateTime?  
  ): Long?  
}
```

다음으로, 영속성 기능을 제공하는 영속성 어뎁터를 구현하자.
애플리케이션에 필요한 LoadAccountPort 와 UpdateAccountStatePort 두 개의 포트를 구현했다.

```kotlin
package com.example.buckpal.adapter.out.persistence  
  
@Component  
class AccountPersistenceAdapter(  
  private val accountRepository: AccountRepository,  
  private val activityRepository: ActivityRepository,  
  private val accountMapper: AccountMapper  
) : LoadAccountPort, UpdateAccountStatePort {  
  
  @Override  
  override fun loadAccount(  
    accountId: Account.AccountId,  
    baselineDate: LocalDateTime  
  ): Account {  
    val account: AccountJpaEntity = accountRepository.findById(accountId.value)  
      .orElseThrow { EntityNotFoundException() }  
  
    val activities = activityRepository.findByOwnerSince(  
      ownerAccountId = accountId.value,  
      since = baselineDate  
    )  
  
    val withdrawalBalance = orZero(  
      activityRepository.getWithdrawalBalanceUntil(  
        accountId = accountId.value,  
        until = baselineDate  
      )  
    )  
  
    val depositBalance = orZero(  
      activityRepository.getDepositBalanceUntil(  
        accountId = accountId.value,  
        until = baselineDate  
      )  
    )  
  
    return accountMapper.mapToDomainEntity(  
      account = account,  
      activities = activities,  
      withdrawalBalance = withdrawalBalance,  
      depositBalance = depositBalance  
    )  
  }  
  
  private fun orZero(value: Long?): Long = value ?: 0L  
  
  @Override  
  override fun updateActivities(account: Account) {  
      
    for (activity in account.activityWindow.activities) {  
      if (activity.id == null) {  
        activityRepository.save(  
          accountMapper.mapToJpaEntity(activity)  
        )  
      }  
    }  
  }  
}
```

---

만들면서 배우는 클린 아키텍처 <톰 홈버그>

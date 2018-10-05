## Reward Dining: Course Reference Domain

## 1. Introduction

The labs of this course teach key concepts in the context of a
problem domain. The domain provides a real-world context for
applying the techniques you have learned to develop useful business
applications. This section provides an overview of the domain and
the applications you will be working on within it.

## 2. Domain Overview 

The Domain is called Reward Dining. The idea behind it is that
customers can save money every time they eat at one of the
restaurants participating to the network. For example, Keith would
like to save money for his children's education. Every time he
dines at a restaurant participating in the network, a contribution
will be made to his account which goes to his daughter Annabelle
for college. See the visual illustrating this business process
below:

## 3. Reward Dining Domain Applications

This next section provides an overview of the applications in
the Reward Dining domain you will be working on in this course.

### 3.1. The Rewards Application

The "rewards" application rewards an account for dining at a
restaurant participating in the reward network. A reward takes the
form of a monetary contribution to an account that is distributed
among the account's beneficiaries. Here is how this application is
used:

1. When they are hungry, members dine at participating restaurants
using their regular credit cards.
2. Every two weeks, a file containing the dining credit card
transactions made by members during that period is generated. A
sample of one of these files is shown below:

```
AMOUNT CREDIT_CARD_NUMBER   MERCHANT_NUMBER DATE
----------------------------------------------------------
100.00  1234123412341234    1234567890      12/29/2010
49.67   1234123412341234    0234567891      12/31/2010
100.00  1234123412341234    1234567890      01/01/2010
27.60   2345234523452345    3456789012      01/02/2010
```

3. A standalone `DiningBatchProcessor` application reads this file and submits each Dining record to the rewards application for processing.

#### 3.1.1. Public Application Interface

The `RewardNetwork` is the central interface clients such as the `DiningBatchProcessor` use to invoke the application:

```java
public interface RewardNetwork {
       RewardConfirmation rewardAccountFor(Dining dining);
}
```

A `RewardNetwork` rewards an account for dining by making a monetary contribution to the account that is distributed among the account's beneficiaries. The sequence diagram below shows a client's interaction with the application illustrating this process:

![rewards-application](https://user-images.githubusercontent.com/558905/40526894-67a05060-5fb7-11e8-8d16-cf53f2f57211.png)

In this example, the account with credit card 1234123412341234 is rewarded for a $100.00 dining at restaurant 1234567890 that took place on 12/29/2010. The confirmed reward 9831 takes the form of an $8.00 account contribution distributed evenly among beneficiaries Annabelle and her brother Corgan.

#### 3.1.2. Internal Application implementation

Internally, the `RewardNetwork` implementation delegates to domain objects to carry out a `rewardAccountFor(Dining)` transaction. Classes exist for the two central domain concepts of the application: `Account` and `Restaurant`. A `Restaurant` is responsible for calculating the benefit eligible to an account for a dining. An `Account` is responsible for distributing
the benefit among its beneficiaries as a "contribution".

This flow is shown below:

![rewardnetwork-domainobject-interaction](https://user-images.githubusercontent.com/558905/40526893-6792e89e-5fb7-11e8-825e-32d03e054e10.png)

The `RewardNetwork` asks the `Restaurant` to calculate how much benefit to award, then contributes that amount to the `Account`.

#### 3.1.3. Supporting Reward Network Component

Account and restaurant information are stored in a persistent form inside a relational database. The `RewardNetwork` implementation delegates to supporting data access components called 'Repositories' to load `Account` and `Restaurant` objects from their relational representations. An `AccountRepository` is used to find an `Account` by its credit card number. A `RestaurantRepository` is used to find a `Restaurant` by its merchant number. A `RewardRepository` is used to track confirmed reward transactions for accounting purposes.

The full `rewardAccountFor(Dining)` sequence incorporating these repositories is shown below:

![rewardaccountfordining-sequence](https://user-images.githubusercontent.com/558905/40526895-67ab010e-5fb7-11e8-9499-d0223fbf75bc.png)

## 4. Reward Dining Database Schema

The Reward Dining applications share a database with this schema:

![rewarddining-databaseschema](https://user-images.githubusercontent.com/558905/40526897-67c11c1e-5fb7-11e8-9ae1-0cbff4949eff.png)

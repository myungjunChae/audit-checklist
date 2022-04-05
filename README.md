# 체크리스트

### 기본 확인 사항

- [ ] Overflow & Underflow
  > 0.8버전에서는 Overflow, Underflow 방지가 기본으로 적용됩니다. [(링크)](https://docs.soliditylang.org/en/v0.8.13/080-breaking-changes.html#silent-changes-of-the-semantics)
- [ ] Visibility
  - 모든 함수의 Visibility가 설계된 로직대로 설정되어있는지 확인해야합니다.
  - 특히 public, external 함수의 외부 접근 권한이 적절히 설정되어있는지 확인해야합니다.
- [ ] 컴파일러 경고 제거
- [ ] 이더리움 전송

  - send, transfer의 사용을 지양하고, call을 사용하는 것이 좋습니다. [(링크)](https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/external-calls/#dont-use-transfer-or-send)
  - 또한 call을 사용할 때는, send, transfer처럼 가스비 제한을 통한 재진입공격 방지책이 없기때문에, 재진입공격 방지(internal link)를 준수해야합니다.

- [ ] 저수준 함수 사용
  - 저수준 함수는 일반 함수 호출과 달리 실패하더라도 error를 throw하지 않습니다. 따라서 결과로 받은 bool값을 통해 실패 시, 처리를 해줘야합니다.
  - 저수준 함수 목록 : `address.call()`, `address.callcode()`, `address.delegatecall()`, `address.send()`
    ```
    (bool success, ) = someAddress.call.value(55)("");
    if(!success) {
      // handle failure code
    }
    ```
- [ ] 문제가 있을 수 있는 기능 사용금지

  - 저수준 함수 사용 (call, delegatecall, callcode, inline assembly) 비권장
  - [var](https://solidity-kr.readthedocs.io/ko/latest/types.html#type-deduction) 사용금지

- [ ] 외부 컨트랙트 호출 [(참고)](https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/external-calls/#use-caution-when-making-external-calls)

  - 신뢰할 수 없는 외부 컨트랙트를 호출하는 것은 위헙합니다.
  - 신뢰할 수 없는 계약에 대해서 직관적으로 알 수 있도록 변수 및 함수명에 기재해야합니다.
  - 외부 컨트랙트를 호출할 일이 있다면, interface 뿐만 아니라, 실제 호출부의 로직을 검토해야합니다.
  - 오딧을 받은 신뢰할 수 있는 외부 코드만 사용합니다.
  - 새로운 코드를 작성하는 것을 최소화하고, 라이브러리를 사용합니다.

    ```
     // bad
      Bank.withdraw(100); // 신뢰 여부를 판단할 수 없음

      // good
      UntrustedBank.withdraw(100); // 신뢰할 수 없는 외부 호출
      TrustedBank.withdraw(100); // 외부 호출이지만, 신뢰할 수 있는 배포자가 운영함
    ```

- [ ] Ether Balance 의존성이 있는 코드 [링크](https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/force-feeding/)

  - 컨트랙트의 특정 로직이 컨트랙트가 가진 Ether Balance에 의존하지 말야아합니다.
  - 악의적인 사용자는 revert로 로직이 되돌려지더라도 계약에 Ether를 보낼 수 있으며, 이러한 상황에서 계약의 Ether Balance에 의존성이 있는 코드가 오작동할 수 있습니다.

- [ ] Pull over push [링크](https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/external-calls/#favor-pull-over-push-for-external-calls)

  - 컨트랙트 내에서 민감한 로직(송금, 권한이전)을 처리할 때, 컨트랙트의 한 함수 내에서 해당 프로세스를 전부 처리(push)하는 것이 아닌, 유저가 호출하는 방식(pull)으로 구현해야합니다
  - payment

    - 컨트랙트 내, 함수에서 직접 external call을 통해서, 유저에게 ether를 전송함(push)
    - 컨트랙트에서 특정 유저가 ether를 가져갈 수 있도록 하고, 유저가 함수를 호출하여 ether를 가져감(pull)

  - owner

    - 컨트랙트 내, owner를 변경하는 함수를 실행하여, 특정유저에게 권한을 넘김(push)
    - 컨트랙트에서 특정 유저가 owner를 가져갈 수 있도록 하고, 해당 유저가 함수를 호출하여 owner 권한을 가져감(pull)

  - 이렇게 pull 방식으로 구현하게되면, 민감한 로직의 실패나 오작동을 방지하여, 위험을 줄일 수 있습니다.

- [ ] [재진입](https://www.mk.co.kr/news/economy/view/2020/05/456982/) 공격 방지 확인
  - check-effect-interaction 패턴을 지켰는지 확인해야합니다.
  - (또는) Reentrancy guard를 적용했는지 확인해야합니다.
- [ ] short circuits(합선)

  - 흔히, "쇼트났다"라는 표현이 의미하듯, 작성한 컨트랙트와 상호작용하는 외부 컨트랙트의 호출 실패를 의미합니다.
  - [외부 디펜던시](https://medium.com/@danielque/what-we-learned-from-auditing-the-top-20-erc20-token-contracts-7526ef3b6fb1)
    - 사용하고 있는 외부 디펜던시가 pausable일 경우, paused 상태에서 되어 동작하지않을 수 잇습니다.
    - unverified code는 실제 코드의 로직을 확인할 수 없어, 실패를 초래할 수 있습니다.
  - [call stack depth](https://docs.soliditylang.org/en/v0.8.13/security-considerations.html?highlight=call%20stack#call-stack-depth)
    - 외부

- [ ] Callstack 공격 확인

  > [방식은 아직 잘 모르겠다](https://hackernoon.com/smart-contract-attacks-part-2-ponzi-games-gone-wrong-d5a8b1a98dd8)

  - EIP150을 통해서, IO heavy operation에 대해 가스비를 대폭 상향시켜, callstack의 최대 깊이까지 도달할 수 없게했기에, 민감하게 신경 쓸 필요는 없습니다. (확인 더 필요)
  - [EIP-608: Tangerine Whistle](https://eips.ethereum.org/EIPS/eip-608)

- [ ] 시간 조작

  > 노드 운영자는 어떻게 시간을 조작할까?

  - timestamp는 이론적으로 악의적인 마이너에 의해서 수분까지 조작될 수 있습니다. 중요한 로직이 timestamp에 의존하고 있진 않은지 체크해야합니다.

- [ ] 반올림 오류

  - 나눗셈의 결과는 항상 정수이며 소수부분은 절사됩니다. 그러나 두 연산자가 literals (또는 리터럴 표현식)인 경우 소수부분은 절사되지
  - 반올림 등이 로직에 영향을 미치면 안됩니다.

- [ ] 랜덤

  - 중요한 메커니즘에서 랜덤(RNG)을 구현할 때, 완벽하지않은 무작위성(blockhash, blocknumber)에 의존하면 안됩니다.

- [ ] External/public 함수 파라미터

  - 외부에서 호출가능한 함수가 파라미터에 따라서 내부 로직의 동작이 바뀌는지 확인해야합니다.

- [ ] 무한 루프

- [ ] 변경된 코드 대응

  |  AS-IS  |    TO-BE     |                 참고                 |
  | :-----: | :----------: | :----------------------------------: |
  | suicide | selfdestruct | https://eips.ethereum.org/EIPS/eip-6 |
  |  sha3   |  keccack256  |

- [ ] 인증
  > 말하는 인증 메커니즘이 뭘까
  - tx.origin을 인증 메커니즘으로 사용하면 안된다.

### Fail-over

- [ ] 데이터와 로직을 다루는 컨트랙트가 분리되어있는지 확인합니다. ([프록시 패턴](https://blog.openzeppelin.com/proxy-patterns/))
- [ ] 어떤 상황이 가장 치명적일지 가정해야합니다.
- [ ] 민감한 변수에 대해서 assert 로직이 있는지 체크해야합니다.
- [ ] 인출 등 중요한 로직이 실행되기까지 일정 텀을 두어, 악의적 행동이 발생했을 때, 대응할 수 있도록한다. ([Speed bump](https://consensys.github.io/smart-contract-best-practices/development-recommendations/precautions/speed-bumps/))
- [ ] 최악의 상황에서 로직실행을 멈출 수 있는 circuit breaker가 있는지 확인해야합니다. ([pausable](https://docs.openzeppelin.com/contracts/2.x/api/lifecycle))

### 주의깊게 봐야하는 부분

- [ ] external, public 함수
- [ ] assembly code 및 저수준 함수 호출
- [ ] 특정 유저들에 대한 특권이 있는 로직
- [ ] 특정 조건(타이밍)에 디펜던시가 있는 로직
- [ ] 네트워크 폭주 시, 영향을 받는 로직
- [ ] 이더리움을 전송하는 payable 함수
- [ ] push payment
- [ ] 오딧을 받지않는 가장 최근에 작성된 로직

### Reference

- [Solidity docs 보안 측 고려사항](https://docs.soliditylang.org/en/v0.8.13/security-considerations.html)
- [Decentralized Application Security Project(DASP)](https://dasp.co/#item-1)
- [cryptofindlab/audit-checklist](https://github.com/cryptofinlabs/audit-checklist)
- [스마트컨트랙트 취약점 분류(SWC)](https://swcregistry.io/docs/SWC-100)
- [TechRate Audit Example](https://github.com/TechRate/Smart-Contract-Audits)
- [SmartContract Bad case](https://github.com/crytic/not-so-smart-contracts)
- [Solidity Pattern](https://github.com/fravoll/solidity-patterns)

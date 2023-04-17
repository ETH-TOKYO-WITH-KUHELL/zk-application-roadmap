# Roadmap
- I want to make some improvements on this project
1. **upgrade login function**
2. **group handling function**
3. **try to make good UI**
4. **upgrade NFT contract for group verification**


# Reference site
- [Semaphore official link ](https://semaphore.appliedzkp.org/)
- [Semaphore identities docs ](https://semaphore.appliedzkp.org/)
- [My Medium posting about using Semaphore Protocol ](https://medium.com/@ChainAnatomy/dev-zk-protocol-semaphore-d45127063e0a)

## 1. Upgrade login function

<aside>
💡 The new identity contains two random secret values trapdoor and nulifier , and one public value commitment

</aside>

- private value
    - `trapdoor`
    - `nulifier`
        - these two secret values are similar to Ethereum private keys and are used to generate Semaphore zero-knowledge proofs and authenticate signals
- public value
    - `Identity commitment`
        - identity commitment is similarly to Ethereum addresses
- generate identites

    ```tsx
    import { Identity } from "@semaphore-protocol/identity"
    
    const { trapdoor, nullifier, commitment } = new Identity()
    ```

## 1.1 Upgrade process

### 1.1.1 AS-IS 

```solidity
// solidity contract
mapping(address => mapping(bytes32 => uint)) private users;

function getUserInfo(bytes32 _username) external view returns(uint){
    require(users[msg.sender][_username] != 0, "there is no user");
    return users[msg.sender][_username];
}
```

```typescript
// next.js
const contract = new ethers.Contract(env.FEEDBACK_CONTRACT_ADDRESS, Feedback.abi, signer)
await contract.getUserInfo(formatBytes32String(_name))
```

### 1.1.2 Identifying the Problem
- 알기 전에는 commitment 값이 private key 값인 줄 알았지만, 이게 public key 값이었다.
- 그래서 이 값이 노출되는게 문제가 되는 줄 알았지만 아니었다.
- 그러면 public 값을 통해 본인임을 증명하게 되면 login 하고 바로 proof를 갔는데, 비밀번호 입력 창으로 이동해서 비밀번호 값을 파라미터로 받아서 `new Idenenty('parameter')` 를 통해 나온 값을 setStorage해주면 문제가 해결될 것 같다


### 1.1.3 TO-BE
```tsx
const identity = new Identity("secret-message")
console.log(identity.toString())
// '["8255d...", "62c41..."]'
```

- `'["8255d...", "62c41..."]'` 이 값이 `new Identity("secret-message")` 값이 같은 경우엔 똑같다
- 그러면 이 값을 setStorage해서 사용하면 `nulifier`, `trapdoor` 값을 proof 할 때 사용할 수 있다

### 1.1.4 Something to prove
- `new Identity("secret-message")` 에서 파라미터로 받은 값이 같은 경우 처음에 생성될 때랑 다시 생성할 때와 같은 지 증명 되어야한다
- 같다면 mapping 에 저장한 public value가 joinGroup transaction을 서명할 때 mapping에 저장되었고, 그룹 NFT를 보유함이 증명되었기 때문에 보안이 적용된다
- 만약 mapping에 저장된 `commitment` 값이 노출되더라도, 처음 joinGroup할 때 입력한 `new Identity("secret-message")` 값은 유저 본인만 알기 때문에 상관이 없다
- 그 값까지 알아야하고, 서명했던 지갑의 private key까지 탈취해야 지갑에 연결해서 서명을 통해 인증되어야 password 페이지로 이동한다
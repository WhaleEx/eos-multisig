# WhaleEx鲸交所智能合约多签解决方案
本文用于阐述"WhaleEx鲸交所智能合约多签解决方案"的目的和执行步骤
## 目的
WhaleEx从设计之初的目标就是做一个没有作恶能力的去中心化交易所, 一直以来, 所有WhaleEx智能合约(whaleextrust)上的资产转出, 都是由该合约的active key(whaleextrust@eosio.code)来执行的. 鉴于我们的初衷, 也为了更好地保护用户的资产, 我们决定升级合约安全级别, 让WhaleEx和EOS主网节点多签来确保用户的资产安全. 
## 优势
该方案执行后, WhaleEx智能合约(whaleextrust)的
  ### 1. owner key: 
  由EOS帐户whaleex.com和8家EOS主网节点(eosasia11111, eosbeijingbp, eosbixinboot, eoscannonchn, eosiomeetone, eoslaomaocom, eosstorebest, jedaaaaaaaaa)共同多签, 门限42, 其中whaleex.com(40), 其他节点(2), 每次执行必须要有1家以上的EOS主网节点认可, 才能执行. 
  ### 2. active key: 
  只有合约代码, 即whaleextrust@eosio.code有执行权限. 没有人可以控制active key. 
  ### 3. 多签执行的前提: 
    a. 只有升级智能合约的transaction才能被认可, 即set_code, set_abi
    b. 每次智能合约升级, 都会由一家主网节点认可的审计方(慢雾科技)审计完成, 并提供hash值
    c. 节点方与whaleex.com通过多签方案中包含的升级内容, 生成hash值, 与#b中审计方提供的hash值校验通过
    以上a, b, c三步均通过后 多签才会被认可执行. 
## 具体步骤
1. 生成多签trx

    ```cleos set contract whaleextrust contract_dir -d -x 86400 -s -j -p whaleextrust@owner > wal_exchange.json ```
      
     计算wal_exchange.json中code hash的方法，需要安装jq
     
    ```cat wal_exchange.json | jq -r '.actions[0].data' | cut -c 27- | xxd -r -p |openssl dgst -sha256```

2. 生成permission.json

    ```
    [
       {"actor": "eosiomeetone", "permission": "active"},
       {"actor": "eosstorebest", "permission": "active"},
       {"actor": "jedaaaaaaaaa", "permission": "active"},
       {"actor": "eoscannonchn", "permission": "active"},
       {"actor": "eosasia11111", "permission": "active"},
       {"actor": "eosbixinboot", "permission": "active"},
       {"actor": "eosbeijingbp", "permission": "active"},
       {"actor": "eoslaomaocom", "permission": "active"},
       {"actor": "whaleex.com", "permission": "active"}
    ]
    ```

3. 提交议案

    ```cleos multisig propose_trx whaleex permission.json wal_exchange.json -p walexecutors```

4. 由合约审计方提供该版本的code hash

5. 多签方审查议案

    ```cleos multisig review walexecutors whaleex | jq '.transaction.actions[0].data.code'|xxd -r -p |openssl dgst -sha256```
    
    查看得到的code hash 是否和合约审计方提供的code hash一致

6. 多签方批准议案

    ```cleos multisig approve walexecutors whaleex '{"actor": "whaleex.com", "permission": "active"}' -p whaleex.com```
    
    ```cleos multisig approve walexecutors whaleex '{"actor": "eoscannonchn", "permission": "active"}' -p eoscannonchn```

7. 执行议案

    ```cleos multisig exec walexecutors whaleex -p walexecutors```

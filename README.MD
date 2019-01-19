
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


## How this Dapp works(all steps are based on MacOS)
1. The tools that we used are 'npm', 'yarn', 'ganache', so please install these tools firstly.
2. The contract compiling and migrating tool is truffle, the commond is following:
    sudo npm install -g truffle
    sudo yarn global add truffl
3. Use "ganache" to create our private chain:
    - open the 'ganache' software, and choose the 'quikestart'
    - click the 'setting' button in the upper right corner
    - in the 'workspace' page, click 'add project' and select file './truffle-config.js'
    - in the 'server' page, change the port to 8545
    - click 'save and restart' in the upper right corner
4. Get the contract addtress:
    - select'contracts' tab in the topest and deploy the 'crowdfunding'
    - copy the address of the contract
    - open file './crowdfunding/src/api/contract.ts', find the 7th line, 
      and replace this addtress with the previous one, which should start with '0x'.
5. Compile and migrate the contract:
    - locate to the crowdfunding-dapp-master, and open the terminal
    - run: truffle compile 
           truffle migrate
    - copy the file './build/contracts/CrowdFunding.json' and paste it to './crowdfunding/src/api/'
6. Run the front-end server and the web
    - relocate to the './crowdfunding/' and run the terminal
    - run: yarn
           sudo yarn serve
7. Test the local webpage:
    - go to the 'localhost:8080'
    - add the account to the wallet, and connect to the 'localhost:8545' to get the ETH
    - Test whatever you want

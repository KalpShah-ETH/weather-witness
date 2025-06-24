### [H-1] The ```WeatherNft:::performUpkeep``` Function Has Missing Access Control.

**Description:** 
The ```WeatherNft:::performUpkeep``` Function Has Missing Access Control.The ```WeatherNft:::performUpkeep``` should 
Only Be Called By The Chainlink Nodes and owner of Token(NFT) .Anyone Can Call ```performUpkeep``` Function
And can Update The States.The Documentation Says That ```performUpkeep``` Function Should Only be Called By 
ChainLink Keepers For Those Users Who Had registered For Automation.


**Impact:** 
Due To Missing Access Control In ```WeatherNft:::performUpkeep``` Function.Any  Malicious User/Attacker  Can Call the Function repeatedly and can update The malicious States and leads to unnecessary gas usage.

**Proof of Concept:**
<details>

```javascript

    function testAnyOneCanCallPerformUpKeepp() public {
        string memory pincode = "125001";
        string memory isoCode = "IN";
        bool registerKeeper = true;
        uint256 heartbeat = 12 hours;
        uint256 initLinkDeposit = 5e18;
        uint256 tokenId = weatherNft.s_tokenCounter();

        vm.startPrank(user);
        linkToken.approve(address(weatherNft), initLinkDeposit);

        bytes32 reqid = weatherNft.requestMintWeatherNFT{
            value: weatherNft.s_currentMintPrice()
        }(pincode, isoCode, registerKeeper, heartbeat, initLinkDeposit);
        vm.stopPrank();

        bytes memory weatherResponse = abi.encode(
            WeatherNftStore.Weather.RAINY
        );
        weatherNft.handleOracleFulfillment(reqid, weatherResponse, "");

        vm.prank(user);
        weatherNft.fulfillMintRequest(reqid);

        vm.prank(user);
        string memory tokenURI = weatherNft.tokenURI(tokenId);

        bytes memory encodedTokenId = abi.encode(tokenId);
        vm.warp(block.timestamp + 12 hours);

        (bool weatherUpdateNeeded, bytes memory performdata) = weatherNft
            .checkUpkeep(encodedTokenId);
        vm.prank(malicioussuser);
        weatherNft.performUpkeep(performdata);
        /*
        Ran 1 test for test/WeatherNftForkTest.t.sol:WeatherNftForkTest
        [PASS] testAnyOneCanCallPerformUpKeepp() (gas: 1130725)
        Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 5.98ms (2.08ms CPU time)
        Ran 1 test suite in 12.55ms (5.98ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
*/
    }

```
</details>

**Recommended Mitigation:**  
For Automatation Registered User,We Wil Use s_keeperRegistry To stop Access From Unauthorized Users

For Manual Users,We Will User Ownerof Token(NFT).

```diff
+require(s_keeperRegistry == msg.sender || ownerOf(tokenId) == msg.sender,"NotAuthorized");
```
Full Code :

<details>

```javascript

 function performUpkeep(bytes calldata performData) external override {
//access control
        require(
            s_keeperRegistry == msg.sender || ownerOf(tokenId) == msg.sender,
            "NotAuthorized"
        );
        uint256 _tokenId = abi.decode(performData, (uint256));
   
        uint256 upkeepId = s_weatherNftInfo[_tokenId].upkeepId;

        s_weatherNftInfo[_tokenId].lastFulfilledAt = block.timestamp;

    
        string memory pincode = s_weatherNftInfo[_tokenId].pincode;
        string memory isoCode = s_weatherNftInfo[_tokenId].isoCode;

        bytes32 _reqId = _sendFunctionsWeatherFetchRequest(pincode, isoCode);
        s_funcReqIdToTokenIdUpdate[_reqId] = _tokenId;

        emit NftWeatherUpdateRequestSend(_tokenId, _reqId, upkeepId);
    }
    

```
</details>

### [H-2] The ```WeatherNft:::tokenURI``` Function Has Missing Access Control.


**Description:** 
The ```WeatherNft:::tokenURI``` Function Has Missing Access Control.The ```WeatherNft:::tokenURI``` should 
 Only Be Called By The Onwer Of That Token(NFT).Anyone Can Call ```tokenURI``` Function And can retrieve the 
 Important Metadata And Sensitive information.Additionally, the _requireOwned function is referenced in the code but is not implemented, resulting in a compilation error and leaving the intended access control unimplemented.

```javacript 

  Undeclared identifier.
   --> src/WeatherNft.sol:293:9:
    |
293 |         _requireOwned(tokenId);
    |         ^^^^^^^^^^^^^

```

**Impact:** 
Due To Undeclared access control ,Any malicious user/attacker can call the ``tokenURI`` function and retreive the Data.

**Proof of Concept:**

Attack Scenario:

Bob mints SUNNY nft with Tokenid(1).

Alice mints Rainy nft with Token(2).

An attacker calls the tokenURI function with tokenId = 1 (Bob's token) and retrieves its metadata without being the owner.

**Recommended Mitigation:**  

Remove The Dead Code 
```diff
-_requireOwned(tokenId);

```
If Protocol is Meant To Check ownership Then Use This From ERC721.

    /**
     * @dev See {IERC721-ownerOf}.
     */
    function ownerOf(uint256 tokenId) public view virtual override returns (address) {
        address owner = _ownerOf(tokenId);
        require(owner != address(0), "ERC721: invalid token ID");
        return owner;
    }

Add This Fuction in ```tokenURI``` Function:

```diff
+require(ownerOf(tokenId) == msg.sender, "caller is not owner");
```
Full Code:
<details>

```javascript
 function tokenURI(
        uint256 tokenId
    ) public view override returns (string memory) {
        
        require(ownerOf(tokenId) == msg.sender, "caller is not owner");

        string memory image = s_weatherToTokenURI[s_tokenIdToWeather[tokenId]];
        bytes memory jsonData = abi.encodePacked(
            '{"name": "Weathear NFT", "user": "',
            Strings.toHexString(_ownerOf(tokenId)),
            '", "image": "',
            image,
            '"}'
        );
        string memory base64TransformedData = Base64.encode(jsonData);
        return string.concat(_baseURI(), base64TransformedData);
    }

```
</details>



### [H-3] The `WeatherNft:::checkUpkeep` Function Has Missing Access Control.

**Description**:

The `WeatherNft:::checkUpkeep` Function Has Missing Access Control.The `WeatherNft:::checkUpkeep` should
Only Be Called By The Chainlink Nodes and owner of Token(NFT) .Anyone Can Call `checkUpkeep` Function
And can Update The States.The Documentation Says That `checkUpkeep` Function Should Only be Called By
ChainLink Keepers For Those Users Who Had registered For Automation.

**Impact**:

Due To Missing Access Control In `WeatherNft:::checkUpkeep` Function.Any Malicious User/Attacker  Can Call the Function repeatedly and can update The malicious States and leads to unnecessary gas usage.

**Proof of Concept**:

```
    function testAnyOneCanCallcheckupkeep() public {
        string memory pincode = "125001";
        string memory isoCode = "IN";
        bool registerKeeper = true;
        uint256 heartbeat = 12 hours;
        uint256 initLinkDeposit = 5e18;
        uint256 tokenId = weatherNft.s_tokenCounter();

        vm.startPrank(user);
        linkToken.approve(address(weatherNft), initLinkDeposit);

        bytes32 reqid = weatherNft.requestMintWeatherNFT{
            value: weatherNft.s_currentMintPrice()
        }(pincode, isoCode, registerKeeper, heartbeat, initLinkDeposit);
        vm.stopPrank();

        bytes memory weatherResponse = abi.encode(
            WeatherNftStore.Weather.RAINY
        );
        weatherNft.handleOracleFulfillment(reqid, weatherResponse, "");

        vm.prank(user);
        weatherNft.fulfillMintRequest(reqid);

        vm.prank(user);
        string memory tokenURI = weatherNft.tokenURI(tokenId);

        bytes memory encodedTokenId = abi.encode(tokenId);
        vm.warp(block.timestamp + 12 hours);
        vm.prank(malicioussuser);
        (bool weatherUpdateNeeded, bytes memory performdata) = weatherNft
            .checkUpkeep(encodedTokenId);
  /*     Ran 1 test for test/WeatherNftForkTest.t.sol:WeatherNftForkTest
        [PASS] testAnyOneCanCallcheckupkeep() (gas: 869190)
        Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 5.16ms (1.66ms CPU time)
        Ran 1 test suite in 22.08ms (5.16ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
*/
    }
```

**Recommended Mitigation**:

```diff
+ +require(s_keeperRegistry == msg.sender || ownerOf(tokenId) == msg.sender,"NotAuthorized");
```

```

 function checkUpkeep(
        bytes calldata checkData
    )
        external
        view
        override
        returns (bool upkeepNeeded, bytes memory performData)
    {
        uint256 _tokenId = abi.decode(checkData, (uint256));

        require(
            s_keeperRegistry == msg.sender || ownerOf(_tokenId) == msg.sender,
            "NotAuthorized"
        );
        if (_ownerOf(_tokenId) == address(0)) {
            upkeepNeeded = false;
        } else {
            upkeepNeeded = (block.timestamp >=
                s_weatherNftInfo[_tokenId].lastFulfilledAt +
                    s_weatherNftInfo[_tokenId].heartbeat);
            if (upkeepNeeded) {
                performData = checkData;
            }
        }
    }

```

### [L-1] Missing Inputs Validation in ```WeatherNft:::constructor```.

**Description:** 

 The ```WeatherNft:::constructor``` takes several parameters in constructor among them some 
are not sanitised or are not validated properly such as, 
 ```address functionsRouter,address _link, address _keeperRegistry,address _keeperRegistrar,_currentMintPrice,_stepIncreasePerMint```.

**Impact:** 
Due To missing inputs validation ,attacker can pass zero addresses which gets updated in 
state variable which leads to malformed states and storage.

**Proof of Concept:**

Deploying the contract with zero addresses:

```javascript

 weathernft = new WeatherNft(
            weathers,
            weatherURI,
            0x0000000000000000000000000000000000000000,
            config,
            0.001 ether,
            0.001 ether,
            0x0000000000000000000000000000000000000000,
            0x0000000000000000000000000000000000000000,
            0x0000000000000000000000000000000000000000,
            50000
        );

```

**Recommended Mitigation:**  

Add require Checks for input validation:

```diff
+require(_link != address(0), "Invalid LINK address");
+require(_keeperRegistry != address(0), "Invalid Keeper Registry");
+require(_keeperRegistrar != address(0), "Invalid Keeper Registrar");
+require(_currentMintPrice > 0, "Invalid Mint Price");
+require(_stepIncreasePerMint > 0, "Invalid Step Increase");

```
FULL CODE:

<details>

```javascript

contract WeatherNft is
    WeatherNftStore,
    ERC721,
    FunctionsClient,
    ConfirmedOwner,
    AutomationCompatibleInterface
{
    using FunctionsRequest for FunctionsRequest.Request;
    using SafeERC20 for IERC20;

    // constructor
    constructor(
        Weather[] memory weathers,
        string[] memory weatherURIs,
        address functionsRouter,
        FunctionsConfig memory _config,
        uint256 _currentMintPrice,
        uint256 _stepIncreasePerMint,
        address _link,
        address _keeperRegistry,
        address _keeperRegistrar,
        uint32 _upkeepGaslimit
    )
        ERC721("Weather NFT", "W-NFT")
        FunctionsClient(functionsRouter)
        ConfirmedOwner(msg.sender)
    {
        require(
            weathers.length == weatherURIs.length,
            WeatherNft__IncorrectLength()
        );

        for (uint256 i; i < weathers.length; ++i) {
            s_weatherToTokenURI[weathers[i]] = weatherURIs[i];
        }

        require(_link != address(0), "Invalid LINK address");
    require(_keeperRegistry != address(0), "Invalid Keeper Registry");
    require(_keeperRegistrar != address(0), "Invalid Keeper Registrar");
    require(_currentMintPrice > 0, "Invalid Mint Price");
    require(_stepIncreasePerMint > 0, "Invalid Step Increase");
        
        s_functionsConfig = _config;
        s_currentMintPrice = _currentMintPrice;
        s_stepIncreasePerMint = _stepIncreasePerMint;
        s_link = _link;
        s_keeperRegistry = _keeperRegistry;
        s_keeperRegistrar = _keeperRegistrar;
        s_upkeepGaslimit = _upkeepGaslimit;
        s_tokenCounter = 1;
    }


```

</details>



### [L-2] Missing Use Of _Safemint in ```WeatherNft::fulfillMintRequest``` Function.

**Description:** 

The ```WeatherNft``` Contract uses _mint Function To mint NFTS.However,
```_safemint``` should be used instead of just _mint function,to ensure
more security.Without ```_safemint``` nfts can get locked permanently in contract
which does not support receiving Them.

**Impact:** 

1.NFTS can get locked in contract permanently,if there is no receiving support.

2.NON-Compliance with ERC721 Standards.

**Proof of Concept:**

Attacker deploys attack contract which does not implement ```IERC721Receiver ``` interface ,
and calls the mint function using attack contract as the msg.sender.

**Recommended Mitigation:**  

Use ```_safemint``` instead of ```_mint```:

```diff
- _mint(msg.sender, tokenId);

+_safeMint(msg.sender,tokenId);

```

### [M-1] Missing input validation in ```WeatherNft::_sendFunctionsWeatherFetchRequest``` Function.

**Description:** 

The ```WeatherNft::_sendFunctionsWeatherFetchRequest``` Function takes inputs of
```_pincode``` and ```_isoCode``` which are not validated properly and leads
to malicious state updates and and passed to ```req.setArgs(_args)``` function.


**Impact:** 
Malicious and extremely large inputs leads to unnecsaary gas consumption and error
in states which depends on ```_pincode``` and ```_isoCode```. 

**Proof of Concept:**

Call The Fuction With malicious inputs:

```javascript

_sendFunctionsWeatherFetchRequest(
     "0000000000000000000000000000000000000000000000000000",
    "INVALID_ISO_CODE_TOO_LONG_TO_BE_VALID"
)

```

**Recommended Mitigation:**

Properly Validate The inputs:

```diff

+require(bytes(_pincode).length>0 && bytes(_pinconde).length<=10>,"too large or too small")
+require(bytes(_isocode).length==2,"invalid iso code")


```


### [L-3] Missing events-logging  for protocol crictical update functions.

**Description:** 
The ```WeatherNft::updateFunctionsGasLimit```,```WeatherNft::updateSubId```,
```WeatherNft::updateSource```,```WeatherNft::updateEncryptedSecretsURL```,
```WeatherNft::updateKeeperGaslimit``` functions lacks important events logging 
which becomes impossible for  frontend to trace the important states updated
of protocol.

**Impact:** 

1.Due to lack of missing events logging ,frontend data tracing and fetching  becomes impossible.

**Proof of Concept:**


**Recommended Mitigation:**  

Add the following code :

```diff

+event functiongaslimitupdated(uint32 oldgaslimit, uint32 newgaslimit);
+event SubIdUpdated(uint64 oldSubId, uint64 newSubId);
+event SourceUpdated(string oldSource, string newSource);
+event EncryptedSecretsURLUpdated(bytes oldEncryptedSecretsURL, bytes newEncryptedSecretsURL);
+event updateKeeperGaslimitupdated(uint32 oldgaslimit,uint32 newgaslimit);
        
 ```

 Revised Code:

 ```

function updateFunctionsGasLimit(uint32 newGaslimit) external onlyOwner {
        emit functiongaslimitupdated (s_functionsConfig.gasLimit,newGaslimit);
        s_functionsConfig.gasLimit = newGaslimit;
      
    }

    function updateSubId(uint64 newSubId) external onlyOwner {
           emit SubIdUpdated(s_functionsConfig.subId, newSubId);
        s_functionsConfig.subId = newSubId;
    }

    function updateSource(string memory newSource) external onlyOwner {
            emit SourceUpdated(s_functionsConfig.source, newSource);
        s_functionsConfig.source = newSource;
    }

    function updateEncryptedSecretsURL(
        bytes memory newEncryptedSecretsURL
    ) external onlyOwner {
        emit EncryptedSecretsURLUpdated(s_functionsConfig.encryptedSecretsURL, newEncryptedSecretsURL);
        s_functionsConfig.encryptedSecretsURL = newEncryptedSecretsURL;
    }

    function updateKeeperGaslimit(uint32 newGaslimit) external onlyOwner {
        emit updateKeeperGaslimitupdated(s_upkeepGaslimit,newGaslimit);
        s_upkeepGaslimit = newGaslimit;
    }


 ```
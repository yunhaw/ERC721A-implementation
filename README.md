# 部署 ERC-721A 並鑄造 NFT 之實作

###### tags: `Notes`
###### Title: 部署 ERC-721A 並鑄造 NFT 之實作
###### Date: 2022.06.15 
###### Update: 2022.06.15
###### Link: https://hackmd.io/@Raheem/B1VKR8DK9
###### GitHub: https://github.com/yunhaw/ERC721A-implementation
###### Author: Raheem
---

## Outline
> 1. 簡介
> 2. 實作
> 2-1. 事前準備
> 2-2. 部署合約
> 2-3. 準備 metadata 並上傳 Pinata(IPFS)
> 2-4. 準備 metadata.json 並上傳 Pinata(IPFS)
> 2-5. metadata.json 與合約連接
> 2-6. 檢查結果
> 2-7. 上傳合約程式碼
> 2-8. 與合約互動
> 3. 合約地址


## 1. 簡介
- 本次實作為透過 ERC-721A 鑄造出 NFT 
- ERC-721A 為 Azuki 所提出的 ERC-721 改良版本
    - [Azuki ERC-721A 官方介紹](https://www.azuki.com/erc721a)
- 與 ERC-721 的差異
    - ERC-721A 較省 Gas
- [ERC-721A 範本](https://github.com/chiru-labs/ERC721A/tree/main/contracts)

## 2. 實作
### 2-1. 事前準備
> 1. 區塊鏈與[以太坊](https://ethereum.org/zh-tw/)基礎概念（[以太坊教科書](https://cypherpunks-core.github.io/products/MasteringEthereum/)）
> 1. 對 Solidity 撰寫的 Smart contract 有初步理解
> 1. 理解 NFT 的組成架構（[Opensea NFT 開發文件](https://docs.opensea.io/docs/metadata-standards)）
> 1. [NFT 基礎概念講解](https://hackmd.io/@Raheem/ry9lPYuXc)
> 1. 熟悉 [RemixIDE](https://remix.ethereum.org) 
> 1. 熟悉 [Etherscan](https://etherscan.io/)
> 1. 熟悉 [Opensea](https://opensea.io/) 二級市場
> 1. 準備好 Metamask，並且有足夠的測試幣(Rinkeby)，並連結 [Opensea Testnets](https://testnets.opensea.io/)
> 1. 註冊並會操作 [Pinata](https://www.pinata.cloud/)(IPFS)
> 1. 準備好欲發行的圖片與 metadata.json

- 接下來會更改智能合約的內容
- 將 [ERC-721A.sol 範本](https://github.com/chiru-labs/ERC721A/blob/main/contracts/ERC721A.sol)複製到 [RemixIDE](https://remix.ethereum.org/)
- 將第 7 行 import './IERC721A.sol'; 刪除，複製 [IERC721A.sol](https://github.com/chiru-labs/ERC721A/blob/main/contracts/IERC721A.sol) 所有的內容貼上取而代之
    - 這邊不透過 import 的方法，而是將 IERC721A.sol 的程式碼全部貼上來

![](https://i.imgur.com/xR8WoSj.png)

- 取代後貼上會如以下所示

![](https://i.imgur.com/iWHZ3xO.png)

- 因為此合約的 mint function 僅提供給合約自身使用，無法被外部調用，所以這邊自行撰寫一個 mint function
    - 以下敘述程式碼的行數可能只是相對位置，因人而異
    - 寫在 contructor 內，使得合約部署後就直接鑄造 NFT，第 391 行
    - 部分程式碼如下，在合約的第 387~392 行
    - 這邊部署合約會直接鑄造 1 顆 NFT，不用再主動 mint

```solidity=
constructor(string memory name_, string memory symbol_) {
    _name = name_;
    _symbol = symbol_;
    _currentIndex = _startTokenId();
    _mint(msg.sender, 1); // 自行加入調用_mint function
}
```
- 也可以撰寫主動 mint 的 function
    - 在第 394 行，加入以下
    - 隨意加入，但若不加，此份合約部署後將無法再 mint 出 NFT

```solidity=
function mint(address to, uint quantity) external {
    _mint(to, quantity);
}
```

- 接著自行撰寫更改 baseURI 的 function，因為原本合約的 baseURI 是寫死的無法更新
    - 在第 612~620 行 
    - baseURI 的用途在於將 NFT 與 metadata.json 做連結

```solidity=
function _baseURI() internal view virtual returns (string memory) {
    return baseURI_; // 這邊記得改成這樣
}

string baseURI_;

function setBaseURI(string memory _str) external {
    baseURI_ = _str;
}
```

## 2-2. 部署合約
- 準備部署此份合約，首先進行編譯
    - 注意編譯器的版本

![](https://i.imgur.com/VmdkvMI.png)

- 進行合約部署(Deploy)
    - 記得選擇 Injected Web3 連結 Metamask 錢包
    - 並選擇此份合約

![](https://i.imgur.com/DmEZ3lP.png)

- 部署成功
    - 可以在 RemixIDE 左側看到此合約的互動功能
    - [在 Etherscan 查看部署是否成功](https://rinkeby.etherscan.io/tx/0x24111becb95dd3ba681990fe5285d1351983af0b9bcb58a8f21eeff32f1cbf28)

![](https://i.imgur.com/eaCpAdE.png)

- 因為此份合約部署時會自動鑄造出 1 顆 NFT，所以直接到 Opensea Testnets 查看 NFT 是否鑄造成功
- NFT 有成功鑄造並且能在 Opensea Testnest 和 Etherscan 上看到，但會發現並沒有圖片以及 NFT 的名稱與屬性等相關資訊
- 接下來要賦予 NFT 圖片、名稱以及屬性等資訊

![](https://i.imgur.com/aT4Nvey.png)

## 2-3. 準備 metadata 並上傳 Pinata(IPFS)

- 建立兩個資料夾，檔名隨意
    - 一個存放：metadata（圖片檔）
    - 另一個存放：metadata.json（描述檔）

![](https://i.imgur.com/WQdW62T.png)

- 第一個資料夾：放圖片檔
    - 放 NFT 的圖片
    - 圖片命名為：0.png（副檔名一定要保留）

![](https://i.imgur.com/tSYj7Ci.png)


- 接著打開並登入 Pinata，將有**圖片檔的資料夾**上傳
    - 檔名隨意，如下圖所示

![](https://i.imgur.com/4CI2Dok.png)

- 成功上傳資料夾至 IPFS

![](https://i.imgur.com/25pbpEm.png)

- 上傳成功後，可點擊該資料夾進入查看
    - 這就是 NFT 圖片的 IPFS 地址，稍後需要加入 metadata.json 檔內，NFT 才看能透過 json 檔導向圖片地址
    - 將網址列的地址複製起來，並在末端加入/0.png，如以下所示
```json=
https://gateway.ipfs.io/ipfs/QmfZUDAmkUAf5R63WnKTosG2zEEYTmE9ubnZSwLJNsUkjL/0.png 
```

![](https://i.imgur.com/MlTDLlN.png)

## 2-4. 準備 metadata.json 並上傳 Pinata(IPFS)

- 第二個資料夾：放JSON描述檔
    - 只要建立一個文字檔，再將副檔名改為 json 即可
    - 描述檔命名為 0.json

![](https://i.imgur.com/HBC1tjk.png)

- 接著用 Visual Studio 開啟後，在 **image 屬性**寫入以下內容
    - 或是任何文字編輯器
    - 編輯完 json 檔內容後記得要把副檔名刪除，只保留檔名 0，如上圖所示
    - 此處的 image 就是前一步驟上傳 Pinata 的圖片的地址

```json=
{
    "description": "Just for test 20220615", 
    "image": "https://gateway.ipfs.io/ipfs/QmfZUDAmkUAf5R63WnKTosG2zEEYTmE9ubnZSwLJNsUkjL/0.png",
    "name": "Raheem test #0",
    "attributes": [
        {
            "trait_type": "Similiar",
            "value": "0%"
        }
    ] 
}
```

![](https://i.imgur.com/vTxaOZm.png)

- 接著將含有 **metadata.json 的資料夾**上傳至 IPFS，方法同前述上傳圖片資料夾
    - 檔名隨意

![](https://i.imgur.com/OYD9mz0.png)

- 成功上傳資料夾

![](https://i.imgur.com/5MNpE8o.png)

- 上傳成功後，可點擊進入查看
    - 將網址列的地址複製起來，如以下所示
    - NFT 會導向此 json 描述檔
    - 該描述檔可以定義此 NFT 的名稱、編號與屬性等資料，並導向 metadata 的地址
```json=
https://gateway.pinata.cloud/ipfs/QmXsFzGSWwQHB72dveUzMXdGASM6SNgnFwfc8tRiMTMzy4
```

![](https://i.imgur.com/nu5iUMo.png)

## 2-5. metadata.json 與合約連接

- 完成將圖片與描述檔上傳 IPFS 後，回到 RemixIDE，對合約進行互動，將 metadata.json 與合約進行串接，賦予鑄造出的 NFT 有定義與圖片等資訊
    - 完成此步驟才會在 Opensea 等二級市場上看到 NFT 的圖片等資訊

- 在 setBaseURI 功能內貼上 metadata.json 在 IPFS 的地址
    - 記得在地址末端加上/，否則會導向錯誤，如以下所示
```json=
https://gateway.pinata.cloud/ipfs/QmXsFzGSWwQHB72dveUzMXdGASM6SNgnFwfc8tRiMTMzy4/
```
![](https://i.imgur.com/QDchUzt.png)

- 交易完成後，可以呼叫 tokenURI 功能查看剛剛的寫入是否成功，若成功會 return 剛剛寫入的地址

![](https://i.imgur.com/IVB0DO0.png)

## 2-6. 檢查結果

- 到 Opensea Testnets 上查看是否成功
    - 記得手動點選 Refresh 更新 metadata
    - Opensea 有時網路會擁塞，但通常 10 分鐘以內就會出現

- NFT 的 metadata 更新成功

![](https://i.imgur.com/i2ViN7h.png)

- 頁面往下，查看該合約的 Details 可以看到相關資訊
    - 點擊該合約地址

![](https://i.imgur.com/ie5jKlY.png)

- 可以在 Etherscan 上看到該合約完整的鏈上資訊

![](https://i.imgur.com/Q47E74W.png)

## 2-7. 上傳合約程式碼

- 再來要上傳此份合約程式碼並通過驗證
    - 以後可以直接在 Etherscan 上進行合約互動，不用再透過 RemixIDE

- 點選 Contract，來到以下頁面
    - 並點擊 Verify and Publish

![](https://i.imgur.com/4rJhmBo.png)

- 填好相關資訊，如下圖所示
    - 若以下資訊與當初使用 RemixIDE 部署合約時的環境不同的話，稍後驗證會出錯

![](https://i.imgur.com/3e7a2b3.png)

- 貼上合約程式碼

![](https://i.imgur.com/B63K6iN.png)

- 點選 Verify and Publish

![](https://i.imgur.com/yHlzyez.png)

- 驗證成功

![](https://i.imgur.com/H6hWJae.png)

- 再回到前一頁，可以看到 Contract 按鈕的右上方出現綠色勾勾

![](https://i.imgur.com/myfialD.png)

## 2-8. 與合約互動

- 在 Etherscan 查看此份合約，點擊有綠色勾勾的 Contract，即可看到開源的合約程式碼內容

![](https://i.imgur.com/KhyK0jd.png)

- 點選上方 Read Contract 按鈕
    - 想查看此份合約鑄造出的某 NFT 就呼叫 tokenURI ，並輸入該 NFT 的 tokenID，這邊輸入 0，點擊 Query
    - 同一份合約所鑄造出的 NFT 的編號通常都是 0、1、2、3、...，以此類推

![](https://i.imgur.com/723u1If.png)

- 導向該地址即可看到 NFT #0 的 metadata.json 內容

![](https://i.imgur.com/e8wBn4O.png)

- 也可以導向該 NFT 的圖片位置

![](https://i.imgur.com/JExDpY6.png)

- 後續想要 mint 等與合約互動，可以點選 Write contract 進行操作

![](https://i.imgur.com/yfT4DjG.png)


## 3. 合約地址

- 此 NFT 的合約地址：[0x0238243FED074fc3AFfF7e935e9416F190952D1e](https://rinkeby.etherscan.io/address/0x0238243FED074fc3AFfF7e935e9416F190952D1e)
- [在 Opensea Testnets 查看此 NFT](https://testnets.opensea.io/assets/rinkeby/0x0238243fed074fc3afff7e935e9416f190952d1e/0)
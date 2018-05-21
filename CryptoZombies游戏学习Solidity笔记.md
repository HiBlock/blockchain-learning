作者：倔强的小红军  
注释：这是学习https://cryptozombies.io时的笔记  
1. 函数的写法  
    ```
    function createZombies(string _str, uint _id) public pure returns(result){
    
    }
    ```
    returns后带括号写返回的参数名

2. 修饰符
- public/private限定函数可以被所有人调用/只能在合约内部使用
- internal/external限定函数函数可以被继承合约调用/只能从外部被调用
- pure/view/留空可以限定函数不使用该DAPP的任何数据（不看也不写）/只查看该DAPP的数据保证不修改/可以修改合约数据
- modifier修饰符，用于自定义其对函数的约束逻辑。在函数之前运行，常用来添加require检查是否满足限制条件，如下展示了自定义参数的modifier，其定义形式跟函数一样：
    ```
    // 存储用户年龄的映射
    mapping (uint => uint) public age;
    
    // 限定用户年龄的修饰符
    modifier olderThan(uint _age, uint _userId) {
      require(age[_userId] >= _age);
      _;
    }
    
    // 必须年满16周岁才允许开车 (至少在美国是这样的).
    // 我们可以用如下参数调用`olderThan` 修饰符:
    function driveCar(uint _userId) public olderThan(16, _userId) { //modifier后也要记得传递参数
      // 其余的程序逻辑
    }
    ```
    modifier修饰符的最后一行为`_;`，表示修饰符调用结束后返回，并执行调用函数余下的部分。  一个常用的modifier叫onlyOwner，它能达到禁止第三方修改我们的合约的同时，留个后门给咱们自己去修改的目的。使用方法为先集成ownable合约。注意onlyOwner表明只能合约的作者调用，而合约的调用者不能调用该函数。 
- payable 一种可以接收以太的特殊函数。可以用于向一个合约要求支付一定的钱来运行一个函数。

# 第三章 高级Solidity编程
这节介绍在以太坊上的DApp与其它程序真正的区别之处
1. 在你把智能协议传上以太坊之后，它就变得不可更改, 这种永固性意味着你的代码永远不能被调整或更新。
因此需要给可能需要修改的参数预留修改接口。

2. 两种存储方式 storage/memory,前者会写入区块链，后者只是临时变量  
默认的函数参数，包括返回的参数，是memory。  
默认的局部变量是一个指向storage的指针。  
状态变量，合约内声明的公有变量。是被分配了空间的storage型数据。  
memory与storage之间的赋值，具体请看[这里](http://me.tryblockchain.org/solidity-data-location.html)


3. 另一个使得Solidity编程语言与众不同的特征：
    ```
      用户想要每次执行你的 DApp 都需要支付一定的 gas（用以太币购买，因此，用户每次跑 DApp 都得花费以太币）。 
    
    ```
    为什么要这样？
    ```  
        用户想要每次执行你的 DApp 都需要支付一定的 gas（用以太币购买，因此，用户每次跑 DApp 都得花费以太币）。 
    ```
    节省`gas`的方法：  
    - 结构封装 （Struct packing）  
    uint即为uint256，另外还有uint32，uint64等，
    因为存储数据比做个运算贵得多，所以当 uint 定义在一个 struct 中的时候，尽量使用最小的整数子类型以节约空间。 并且把同样类型的变量放一起（即在 struct 中将把变量按照类型依次放置），这样 Solidity 可以将存储空间最小化。
    - 利用view函数节省gas  
    view 函数不会真正改变区块链上的任何数据——它们只是读取。  
    用 view 标记一个函数，意味着运行这个函数只需要查询你的本地以太坊节点，而不需要在区块链上创建一个事务（事务需要运行在每个节点上，因此花费 gas）。  
    因此在所能只读的函数上标记上表示“只读”的“external view 声明，就能为你的玩家减少在 DApp 中 gas 用量。
    - `storage`(存储)非常昂贵  
        - 大多数编程语言中，遍历大数据集合都是昂贵的。但在 Solidity 中，使用一个标记了external view的函数，遍历比 storage 要便宜太多，因为 view 函数不会产生任何花销。  
        - 在数组后面加上 memory关键字，表明这个数组是仅仅在内存中创建，不需要写入外部存储，并且在函数调用结束时它就解散了。如：
        
            ```
              uint[] memory values = new uint[](3);
            
            ```
    
        - 其实就是将程序用到的数据范围严格限定，避免不必要的存储，以节约gas 
    
    在大多数编程语言中，遍历大数据集合都是昂贵的。但是在 Solidity 中，使用一个标记了external view的函数，遍历比 storage 要便宜太多，因为 view 函数不会产生任何花销。 （gas可是真金白银啊！）。


4. Solidity中的时间
unix时间戳，Solidity中还包含seconds,minutes,hours,days,weeks,years 等时间单位，分别会转换为相应的秒数存入uint中。
now变量（uint256类型），固定返回当前时间

5. Solidity的继承，可以像传统一样定义is a关系，但好像还可以用于分割文件，避免一个文件中的逻辑过于复杂。
    ```
    contract cat is animal, creature{
        
    }
    ```

# 第五章 代币和交易
1. #### 什么是代币？  
    一个 代币 在以太坊基本上就是一个遵循一些共同规则的智能合约——即它实现了所有其他代币合约共享的一组标准函数，例如 transfer(address _to, uint256 _value) 和 balanceOf(address _owner).  
    在智能合约内部，通常有一个映射， mapping(address => uint256) balances，用于追踪每个地址还有多少余额。  
    所以基本上一个代币只是一个追踪谁拥有多少该代币的合约，和一些可以让那些用户将他们的代币转移到其他地址的函数。  
    #### 什么是ERC20?  
    ERC20事实上是一个标准，满足这个标准的代币就叫ERC20代币，他们共享具有相同名称的同一组函数，相互之间可以以相同的方式进行交互。  
    类似的标准还有ERC721等，它规定每个代币都被认为是唯一且不可分割的。只能以整个单位交易它们，并且每个单位都有唯一的 ID。  
    使用标准的优势在于：在我们的合约中不必再实现交易逻辑。并且其他人可以为加密可交易的ERC721资产搭建一个交易所平台

2. 库的使用
    ```
    import "./safemath.sol";
    
    using SafeMath for uint256; // 这句写在contract中
    ```
    SafeMath库是OpenZeppelin建立的用来防止数据溢出的数学运算库，using可以自动把库的所有方法添加给某个数据类型，这样就不用每次都c = add(a,b)而可以简写成c = a.add(b)了。

3. 注释风格
    Solidity 社区所使用的一个标准是使用一种被称作 natspec 的格式，看起来像这样：
    ```
    /// @title 一个简单的基础运算合约
    /// @author H4XF13LD MORRIS 💯💯😎💯💯
    /// @notice 现在，这个合约只添加一个乘法
    contract Math {
      /// @notice 两个数相乘
      /// @param x 第一个 uint
      /// @param y  第二个 uint
      /// @return z  (x * y) 的结果
      /// @dev 现在这个方法不检查溢出
      function multiply(uint x, uint y) returns (uint z) {
        // 这只是个普通的注释，不会被 natspec 解释
        z = x * y;
      }
    }
    ```
    @title（标题） 和 @author （作者）很直接了.
    
    @notice （须知）向 用户 解释这个方法或者合约是做什么的。 @dev （开发者） 是向开发者解释更多的细节。
    
    @param （参数）和 @return （返回） 用来描述这个方法需要传入什么参数以及返回什么值。
    
    注意你并不需要每次都用上所有的标签，它们都是可选的。不过最少，写下一个 @dev 注释来解释每个方法是做什么的。

# 第六章 应用前端和 Web3.js
用以太坊基金发布的 JavaScript 库 —— Web3.js，来为 DApp 创建一个基本的前端界面，和智能合约交互。

1. 什么是web3.js  
    以太坊网络是由节点组成的，每一个节点都包含了区块链的一份拷贝。当你想要调用一份智能合约的一个方法，你需要从其中一个节点中查找并告诉它:
    - 智能合约的地址
    - 你想调用的方法，以及
    - 你想传入那个方法的参数  
    
    以太坊节点只能识别一种叫做JSON-RPC的语言。这种语言直接读起来并不好懂。当你你想调用一个合约的方法的时候，需要发送的查询语句将会是这样的：  
    `{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}`  
    web3.js是以太坊基金发布的 JavaScript 库，封装了查询语句，提供更易用的交互界面。  
    Web3 Provider用来告诉我们的代码应该和哪个节点进行交互（就好比是传统的 Web 应用程序中为你的 API 调用设置远程 Web 服务器的网址）  
    可以运行你自己的以太坊节点来作为 Provider，也可以用第三方的服务：[Infura](https://infura.io/)。它维护了很多以太坊节点并提供了一个缓存层来实现高速读取。用 Infura作为节点提供者，你可以不用自己运营节点就能很可靠地向以太坊发送、接收信息。  
    MetaMask：浏览器中的身份（私钥）管理插件，让你在浏览器中访问DAPP，而不需要运行完整的以太坊代码。

2. 使用方法:  
    - Web3.js 需要两个东西来和你的合约对话: 它的 地址 和它的 ABI。  
        > ABI（Application Binary Interface）: 意为应用二进制接口，它是以 JSON 格式表示合约的方法，告诉 Web3.js 如何以合同理解的方式格式化函数调用。
    - Web3.js 有两个方法来调用我们合约的函数: call and send.
        > call 用来调用 view 和 pure 函数。它只运行在本地节点，不会在区块链上创建事务。
        > send 将创建一个事务并改变区块链上的数据。你需要用 send 来调用任何非 view 或者 pure 的函数。
    






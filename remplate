/**
 * Created by Yuri Nikiforov.
 * Date: 06.02.2019
 * Time: 20:53
 **/

pragma solidity >=0.4.21 <0.6.0;

contract SafeMath {
    //internals

    function safeMul(uint a, uint b) internal pure returns(uint) {
        uint c = a * b;
        assert(a == 0 || c / a == b);
        return c;
    }

    function safeSub(uint a, uint b) internal pure returns(uint) {
        assert(b <= a);
        return a - b;
    }

    function safeAdd(uint a, uint b) internal pure returns(uint) {
        uint c = a + b;
        assert(c >=a && c >= b);
        return c;
    }
}

contract DatrixoToken is SafeMath {
    /* Public variables of the token */

    string constant public standard = "ERC20";
    string constant public name = "DarixoToken";
    string constant public symbol = "DRT";
    uint8 constant public decimals = 5;
    uint public totalSupply = 40240000000000;
    uint constant public tokensForIco = 20120000000000;
    uint constant public reservedAmount = 20120000000000;
    uint constant public lockedAmount = 15291200000000;
    address public owner;
    /* from this time on tokens may be transfered (after ICO) */
    uint public startTime;
    /* tells if tokens have been burned already */
    bool burned;

    /* This creates an array with all balances */

    /* This balance structure is
    *  account address -> value of sold Tokens - balanceOf
    *  account address -> time of first purchase - firstPurchaseTime
    *  transfer check firstPurchseTime if 0 - free sale
    */
    mapping(address => uint) public balanceOf;
    mapping(address => uint) public firstPurchaseTime;
    /*
    * List of shareholders
    */
    address[] public shareholders;

    /* This allowance structure is
     * seller account -> customer account -> allowance value of Tokens
    */
    mapping(address => mapping(address => uint)) public allowance;


    /* This generates a public event on the blockchain that will notify clients */
    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed _owner, address indexed spender, uint value);
    event Burned(uint amount);


    /* Initializes contract with initial supply tokens to the creator of the contract */
    constructor(address _ownerAddr, uint _startTime) public {
        owner = _ownerAddr;
        startTime = _startTime;
        balanceOf[owner] = totalSupply; // Give the owner all initial tokens
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "You are not contract owner.");
        _;
    }

    modifier afterStartTime() {
        require(now > startTime, "STO is not started.");
        _;
    }

    modifier onlyShareholder() {
        require(balanceOf[msg.sender] > 0, "You are not shareholder.");
        _;
    }

    modifier afterFirstYear() {
        require(firstPurchaseTime[msg.sender] == 0 || now >=firstPurchaseTime[msg.sender] + 365 days, "First year is not expired.");
        _;
    }

    function transfer(address _to, uint _value) public returns(bool success){
        require(msg.sender != _to, "Target address can't be equal source.");
        if (msg.sender == owner) {
            return _firstTransfer(_to, _value);
        } else {
            return _secondTransfer(_to, _value);
        }
    }

    /* First send tokens to shareholder by owner*/
    function _firstTransfer(address _to, uint _value) internal onlyOwner afterStartTime returns(bool success) {
        require(_to != address(0), "Target address is 0x0"); // prevent the owner to spending to address 0x0
        require(balanceOf[_to] == 0, "Target balance not equal 0"); // prevent secondary transfer
        require(safeSub(balanceOf[msg.sender], _value) >= lockedAmount, "Value more then locked amount"); // prevent the owner to spending his share of tokens for company, loyalty program and future financing of the company within the first year
        shareholders.push(_to);
        firstPurchaseTime[_to] = now;
        return _transfer(_to, _value);
    }

    /* Send some of your tokens*/
    function _secondTransfer(address _to, uint _value) internal onlyShareholder afterFirstYear returns(bool success){
        require(_to != address(0), "Target address is 0x0"); // prevent the owner to spending to address 0x0
        require(firstPurchaseTime[_to] == 0, "Target balance has first transfer amount.");
        if (firstPurchaseTime[msg.sender] > 0) {
            delete firstPurchaseTime[msg.sender];
        }
        if (!checkShareholderExist(_to)) {
            shareholders.push(_to);
        }
        return _transfer(_to, _value);
    }

    function checkShareholderExist(address _addr) internal view returns(bool) {
        for (uint i = 1; i < shareholders.length; i++) {
            if (shareholders[i] == _addr) return true;
        }
        return false;
    }

    function _transfer(address _to, uint _value) internal returns(bool success){
        balanceOf[msg.sender] = safeSub(balanceOf[msg.sender], _value); // Subtract from the sender
        balanceOf[_to] = safeAdd(balanceOf[_to], _value); // Add the same to the recipient purchase with _transactionTime
        emit Transfer(msg.sender, _to, _value); // Notify anyone listening that this transfer took place
        return true;
    }

    /*
    * Getter for whole shareholders array, not any array mamber as default getter, which generated complier
    */
    function getShareholdersArray() public view returns(address[] memory) {
        return shareholders;
    }


    /* to be called when STO is closed. burns the remaining tokens except the company share (60360000), the tokens reserved
     * for the bounty/advisors/marketing program (48288000), for the loyalty program (52312000) and for future financing of the company (40240000).
     * anybody may burn the the tokens after ICO ended, but only once (in case the owner holds more tokens in the future).
     * this ensures that the owner will not posses a majority of the tokens. */
    function burn() public onlyOwner afterStartTime {
        // if token have not been burned already and the STO ended
        require(!burned, "Token have been burned already.");
        uint difference = safeSub(balanceOf[owner], reservedAmount);
        balanceOf[owner] = reservedAmount;
        totalSupply = safeSub(totalSupply, difference);
        burned = true;
        emit Burned(difference);

    }

    /**
     * Allows the sto contract to set the traiding start time to an earler point of time.
     * (In case the soft cap has been reached)
     * @param _newStart the new start date
     **/
    function setStart(uint _newStart) public onlyOwner {
        require(_newStart < startTime, "New start time must be earlier current start time.");
        startTime = _newStart;
    }
}

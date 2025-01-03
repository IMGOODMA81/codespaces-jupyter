pragma solidity ^0.4.23;

import "./TRC20.sol";
import "./MinterRole.sol";

/**
 * @title TRC20Detailed token
 * @dev The decimals are only for visualization purposes.
 * All the operations are done using the smallest and indivisible token unit,
 * just as on TRON all the operations are done in sun.
 *
 * Example inherits from basic TRC20 implementation but can be modified to
 * extend from other ITRC20-based tokens:
 * https://github.com/OpenZeppelin/openzeppelin-solidity/issues/1536
 */
contract WINK is TRC20, MinterRole {
    string private _name;
    string private _symbol;
    uint8 private _decimals;

    constructor (string name, string symbol, uint8 decimals) public {
        _name = name;
        _symbol = symbol;
        _decimals = decimals;
    }

    /**
     * @return the name of the token.
     */
    function name() public view returns (string) {
        return _name;
    }

    /**
     * @return the symbol of the token.
     */
    function symbol() public view returns (string) {
        return _symbol;
    }

    /**
     * @return the number of decimals of the token.
     */
    function decimals() public view returns (uint8) {
        return _decimals;
    }

    /**
     * @dev Function to mint tokens
     * @param to The address that will receive the minted tokens.
     * @param value The amount of tokens to mint.
     * @return A boolean that indicates if the operation was successful.
     */
    function mint(address to, uint256 value) public onlyMinter returns (bool) {
        _mint(to, value);
        return true;
    }

    /**
     * @dev Burns a specific amount of tokens.
     * @param value The amount of token to be burned.
     */
    function burn(uint256 value) public {
        _burn(msg.sender, value);
    }

    /**
     * @dev Burns a specific amount of tokens from the target address and decrements allowance
     * @param from address The address which you want to send tokens from
     * @param value uint256 The amount of token to be burned
     */
    function burnFrom(address from, uint256 value) public {
        _burnFrom(from, value);
    }
}pragma solidity ^0.4.23;

import "./ITRC20.sol";
import "./SafeMath.sol";

/**
 * @title Standard TRC20 token (compatible with ERC20 token)
 *
 * @dev Implementation of the basic standard token.
 * https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
 * Originally based on code by FirstBlood: https://github.com/Firstbloodio/token/blob/master/smart_contract/FirstBloodToken.sol
 */
contract TRC20 is ITRC20 {
    using SafeMath for uint256;

    mapping (address => uint256) private _balances;

    mapping (address => mapping (address => uint256)) private _allowed;

    uint256 private _totalSupply;

    /**
     * @dev Total number of tokens in existence
     */
    function totalSupply() public view returns (uint256) {
        return _totalSupply;
    }

    /**
     * @dev Gets the balance of the specified address.
     * @param owner The address to query the balance of.
     * @return An uint256 representing the amount owned by the passed address.
     */
    function balanceOf(address owner) public view returns (uint256) {
        return _balances[owner];
    }

    /**
     * @dev Function to check the amount of tokens that an owner allowed to a spender.
     * @param owner address The address which owns the funds.
     * @param spender address The address which will spend the funds.
     * @return A uint256 specifying the amount of tokens still available for the spender.
     */
    function allowance(
        address owner,
        address spender
    )
    public
    view
    returns (uint256)
    {
        return _allowed[owner][spender];
    }

    /**
     * @dev Transfer token for a specified address
     * @param to The address to transfer to.
     * @param value The amount to be transferred.
     */
    function transfer(address to, uint256 value) public returns (bool) {
        _transfer(msg.sender, to, value);
        return true;
    }

    /**
     * @dev Approve the passed address to spend the specified amount of tokens on behalf of msg.sender.
     * Beware that changing an allowance with this method brings the risk that someone may use both the old
     * and the new allowance by unfortunate transaction ordering. One possible solution to mitigate this
     * race condition is to first reduce the spender's allowance to 0 and set the desired value afterwards:
     * https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
     * @param spender The address which will spend the funds.
     * @param value The amount of tokens to be spent.
     */
    function approve(address spender, uint256 value) public returns (bool) {
        require(spender != address(0));

        _allowed[msg.sender][spender] = value;
        emit Approval(msg.sender, spender, value);
        return true;
    }

    /**
     * @dev Transfer tokens from one address to another
     * @param from address The address which you want to send tokens from
     * @param to address The address which you want to transfer to
     * @param value uint256 the amount of tokens to be transferred
     */
    function transferFrom(
        address from,
        address to,
        uint256 value
    )
    public
    returns (bool)
    {
        _allowed[from][msg.sender] = _allowed[from][msg.sender].sub(value);
        _transfer(from, to, value);
        return true;
    }

    /**
     * @dev Increase the amount of tokens that an owner allowed to a spender.
     * approve should be called when allowed_[_spender] == 0. To increment
     * allowed value is better to use this function to avoid 2 calls (and wait until
     * the first transaction is mined)
     * From MonolithDAO Token.sol
     * @param spender The address which will spend the funds.
     * @param addedValue The amount of tokens to increase the allowance by.
     */
    function increaseAllowance(
        address spender,
        uint256 addedValue
    )
    public
    returns (bool)
    {
        require(spender != address(0));

        _allowed[msg.sender][spender] = (
        _allowed[msg.sender][spender].add(addedValue));
        emit Approval(msg.sender, spender, _allowed[msg.sender][spender]);
        return true;
    }

    /**
     * @dev Decrease the amount of tokens that an owner allowed to a spender.
     * approve should be called when allowed_[_spender] == 0. To decrement
     * allowed value is better to use this function to avoid 2 calls (and wait until
     * the first transaction is mined)
     * From MonolithDAO Token.sol
     * @param spender The address which will spend the funds.
     * @param subtractedValue The amount of tokens to decrease the allowance by.
     */
    function decreaseAllowance(
        address spender,
        uint256 subtractedValue
    )
    public
    returns (bool)
    {
        require(spender != address(0));

        _allowed[msg.sender][spender] = (
        _allowed[msg.sender][spender].sub(subtractedValue));
        emit Approval(msg.sender, spender, _allowed[msg.sender][spender]);
        return true;
    }

    /**
     * @dev Transfer token for a specified addresses
     * @param from The address to transfer from.
     * @param to The address to transfer to.
     * @param value The amount to be transferred.
     */
    function _transfer(address from, address to, uint256 value) internal {
        require(to != address(0));

        _balances[from] = _balances[from].sub(value);
        _balances[to] = _balances[to].add(value);
        emit Transfer(from, to, value);
    }

    /**
     * @dev Internal function that mints an amount of the token and assigns it to
     * an account. This encapsulates the modification of balances such that the
     * proper events are emitted.
     * @param account The account that will receive the created tokens.
     * @param value The amount that will be created.
     */
    function _mint(address account, uint256 value) internal {
        require(account != address(0));

        _totalSupply = _totalSupply.add(value);
        _balances[account] = _balances[account].add(value);
        emit Transfer(address(0), account, value);
    }

    /**
     * @dev Internal function that burns an amount of the token of a given
     * account.
     * @param account The account whose tokens will be burnt.
     * @param value The amount that will be burnt.
     */
    function _burn(address account, uint256 value) internal {
        require(account != address(0));

        _totalSupply = _totalSupply.sub(value);
        _balances[account] = _balances[account].sub(value);
        emit Transfer(account, address(0), value);
    }

    /**
     * @dev Internal function that burns an amount of the token of a given
     * account, deducting from the sender's allowance for said account. Uses the
     * internal burn function.
     * @param account The account whose tokens will be burnt.
     * @param value The amount that will be burnt.
     */
    function _burnFrom(address account, uint256 value) internal {
        // Should https://github.com/OpenZeppelin/zeppelin-solidity/issues/707 be accepted,
        // this function needs to emit an event with the updated approval.
        _allowed[account][msg.sender] = _allowed[account][msg.sender].sub(
            value);
        _burn(account, value);
    }
}
pragma solidity ^0.4.23;

/**
 * @title Roles
 * @dev Library for managing addresses assigned to a Role.
 */
library Roles {
    struct Role {
        mapping (address => bool) bearer;
    }

    /**
     * @dev give an account access to this role
     */
    function add(Role storage role, address account) internal {
        require(account != address(0));
        require(!has(role, account));

        role.bearer[account] = true;
    }

    /**
     * @dev remove an account's access to this role
     */
    function remove(Role storage role, address account) internal {
        require(account != address(0));
        require(has(role, account));

        role.bearer[account] = false;
    }

    /**
     * @dev check if an account has this role
     * @return bool
     */
    function has(Role storage role, address account) internal view returns (bool) {
        require(account != address(0));
        return role.bearer[account];
    }
}pragma solidity ^0.4.23;

import "./Roles.sol";

contract MinterRole {
    using Roles for Roles.Role;

    event MinterAdded(address indexed account);
    event MinterRemoved(address indexed account);

    Roles.Role private _minters;

    constructor () internal {
        _addMinter(msg.sender);
    }

    modifier onlyMinter() {
        require(isMinter(msg.sender));
        _;
    }

    function isMinter(address account) public view returns (bool) {
        return _minters.has(account);
    }

    function addMinter(address account) public onlyMinter {
        _addMinter(account);
    }

    function renounceMinter() public {
        _removeMinter(msg.sender);
    }

    function _addMinter(address account) internal {
        _minters.add(account);
        emit MinterAdded(account);
    }

    function _removeMinter(address account) internal {
        _minters.remove(account);
        emit MinterRemoved(account);
    }
}// File: contracts/common/misc/ERCProxy.sol

/*
 * SPDX-License-Identitifer:    MIT
 */

pragma solidity ^0.5.2;

// See https://github.com/ethereum/EIPs/blob/master/EIPS/eip-897.md

interface ERCProxy {
    function proxyType() external pure returns (uint256 proxyTypeId);
    function implementation() external view returns (address codeAddr);
}

// File: contracts/common/misc/DelegateProxyForwarder.sol

pragma solidity ^0.5.2;

contract DelegateProxyForwarder {
    function delegatedFwd(address _dst, bytes memory _calldata) internal {
        // solium-disable-next-line security/no-inline-assembly
        assembly {
            let result := delegatecall(
                sub(gas, 10000),
                _dst,
                add(_calldata, 0x20),
                mload(_calldata),
                0,
                0
            )
            let size := returndatasize

            let ptr := mload(0x40)
            returndatacopy(ptr, 0, size)

            // revert instead of invalid() bc if the underlying call failed with invalid() it already wasted gas.
            // if the call returned error data, forward it
            switch result
                case 0 {
                    revert(ptr, size)
                }
                default {
                    return(ptr, size)
                }
        }
    }
    
    function isContract(address _target) internal view returns (bool) {
        if (_target == address(0)) {
            return false;
        }

        uint256 size;
        assembly {
            size := extcodesize(_target)
        }
        return size > 0;
    }
}

// File: contracts/common/misc/DelegateProxy.sol

pragma solidity ^0.5.2;



contract DelegateProxy is ERCProxy, DelegateProxyForwarder {
    function proxyType() external pure returns (uint256 proxyTypeId) {
        // Upgradeable proxy
        proxyTypeId = 2;
    }

    function implementation() external view returns (address);
}

// File: contracts/common/misc/UpgradableProxy.sol

pragma solidity ^0.5.2;


contract UpgradableProxy is DelegateProxy {
    event ProxyUpdated(address indexed _new, address indexed _old);
    event OwnerUpdate(address _new, address _old);

    bytes32 constant IMPLEMENTATION_SLOT = keccak256("matic.network.proxy.implementation");
    bytes32 constant OWNER_SLOT = keccak256("matic.network.proxy.owner");

    constructor(address _proxyTo) public {
        setOwner(msg.sender);
        setImplementation(_proxyTo);
    }

    function() external payable {
        // require(currentContract != 0, "If app code has not been set yet, do not call");
        // Todo: filter out some calls or handle in the end fallback
        delegatedFwd(loadImplementation(), msg.data);
    }

    modifier onlyProxyOwner() {
        require(loadOwner() == msg.sender, "NOT_OWNER");
        _;
    }

    function owner() external view returns(address) {
        return loadOwner();
    }

    function loadOwner() internal view returns(address) {
        address _owner;
        bytes32 position = OWNER_SLOT;
        assembly {
            _owner := sload(position)
        }
        return _owner;
    }

    function implementation() external view returns (address) {
        return loadImplementation();
    }

    function loadImplementation() internal view returns(address) {
        address _impl;
        bytes32 position = IMPLEMENTATION_SLOT;
        assembly {
            _impl := sload(position)
        }
        return _impl;
    }

    function transferOwnership(address newOwner) public onlyProxyOwner {
        require(newOwner != address(0), "ZERO_ADDRESS");
        emit OwnerUpdate(newOwner, loadOwner());
        setOwner(newOwner);
    }

    function setOwner(address newOwner) private {
        bytes32 position = OWNER_SLOT;
        assembly {
            sstore(position, newOwner)
        }
    }

    function updateImplementation(address _newProxyTo) public onlyProxyOwner {
        require(_newProxyTo != address(0x0), "INVALID_PROXY_ADDRESS");
        require(isContract(_newProxyTo), "DESTINATION_ADDRESS_IS_NOT_A_CONTRACT");

        emit ProxyUpdated(_newProxyTo, loadImplementation());
        
        setImplementation(_newProxyTo);
    }

    function updateAndCall(address _newProxyTo, bytes memory data) payable public onlyProxyOwner {
        updateImplementation(_newProxyTo);

        (bool success, bytes memory returnData) = address(this).call.value(msg.value)(data);
        require(success, string(returnData));
    }

    function setImplementation(address _newProxyTo) private {
        bytes32 position = IMPLEMENTATION_SLOT;
        assembly {
            sstore(position, _newProxyTo)
        }
    }
}

// File: contracts/staking/stakeManager/StakeManagerProxy.sol

pragma solidity ^0.5.2;


contract StakeManagerProxy is UpgradableProxy {
    constructor(address _proxyTo) public UpgradableProxy(_proxyTo) {}
}https://tronscan.org/#/contract/TEpjT8xbAe3FPCPFziqFfEjLVXaw9NbGXj/code?file=F1L1{"name":"Superchain Token List","timestamp":"2024-10-30T03:50:07.492Z","version":{"major":10,"minor":0,"patch":152},"logoURI":"https://ethereum-optimism.github.io/optimism.svg","tokens":[{"chainId":8453,"address":"0xc5fecC3a29Fb57B5024eEc8a2239d4621e111CBE","name":"1INCH Token","symbol":"1INCH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/1INCH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x681A09A902D9C7445b3B1Ab282C38D60c72F1f09","name":"AlphaKEK.AI","symbol":"AIKEK","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/AIKEK/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x97c806e7665d3AFd84A8Fe1837921403D59F3Dcc","name":"Artificial Liquid Intelligence Token","symbol":"ALI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ALI/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x13F4196cC779275888440b3000AE533BbBbC3166","name":"Alongside Crypto Market Index","symbol":"AMKT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/AMKT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x7A2C5e7788E55Ec0a7ba4aEeC5B3da322718Fb5e","name":"Apu Apustaja","symbol":"APU","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/APU/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x998b3Fd0998bffF66D28aB03DfE417FC3B884188","name":"ARIANEE","symbol":"ARIA20","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ARIA20/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1C9Fa01e87487712706Fb469a13bEb234262C867","name":"ARPA Token","symbol":"ARPA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ARPA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x74F804B4140ee70830B3Eef4e690325841575F89","name":"Fetch","symbol":"FET","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ASI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x58D75a1c4477914f9a98A8708fEaeD1DbE40b8a3","name":"AthenaDAO Token","symbol":"ATH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ATH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1509706a6c66CA549ff0cB464de88231DDBe213B","name":"Aura","symbol":"AURA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/AURA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x223738a747383d6F9f827d95964e4d8E8AC754cE","name":"Aura BAL","symbol":"auraBAL","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/auraBAL/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x9B700B043e9587ddE9a0c29A9483e2F8FA450d54","name":"AxonDAO Governance Token","symbol":"AXGT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/AXGT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4158734D47Fc9692176B5085E0F52ee0Da5d47F1","name":"Balancer","symbol":"BAL","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/BAL/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf1095aa7DC127ed45441aCfb742694d6a0261b00","name":"Bepro Token","symbol":"BEPRO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/BEPRO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2a06A17CBC6d0032Cac2c6696DA90f29D39a1a29","name":"HarryPotterObamaSonic10Inu","symbol":"BITCOIN","decimals":8,"logoURI":"https://ethereum-optimism.github.io/data/BITCOIN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xddB293BB5C5258F7484A94a0fBd5c8B2F6E4e376","name":"Brickken","symbol":"BKN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/BKN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x30136B90e532141FeD006c61105cff3668b5c774","name":"BlueSwap","symbol":"BLUE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/BLUE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1F9bD96DDB4Bd07d6061f8933e9bA9EDE9967550","name":"Boba Token","symbol":"BOBA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/BOBA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x0956CB4A1D8924680FEb671d2E4a122E2114313e","name":"BOOK OF MEME","symbol":"BOME","decimals":6,"logoURI":"https://ethereum-optimism.github.io/data/BOME/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xd9aAEc86B65D86f6A7B5B1b0c42FFA531710b6CA","name":"USD Base Coin","symbol":"USDbC","decimals":6,"logoURI":"https://ethereum-optimism.github.io/data/BridgedUSDC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE678215723A44647F4729E92E0032e1634105CAb","name":"Burn","symbol":"BURN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/BURN/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf0f326af3b1Ed943ab95C29470730CC8Cf66ae47","name":"Burn Wrapped AJNA","symbol":"bwAJNA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/bwAJNA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2Ae3F1Ec7F1F5012CFEab0185bfc7aa3cf0DEc22","name":"Coinbase Wrapped Staked ETH","symbol":"cbETH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/cbETH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD988824d7fA1998D0ABBab2F79A82aF762886caC","name":"CropBytes","symbol":"CBX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CBX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2FA824bB2e06D6caC7DBAb00bE37Dc0F3D440E8b","name":"One Cent Coin","symbol":"🇺🇸","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CENT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x20b048fA035D5763685D695e66aDF62c5D9F5055","name":"Biochar","symbol":"CHAR","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CHAR/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf88Cf71B891a89A1CB02ec17ce3aD80e5Ae31e1b","name":"Cigarette Token","symbol":"CIG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CIG/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE2B21D4684b2bA62F3BE1FE286eacb90D26E394d","name":"CryptoOracle Collective","symbol":"COC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/COC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x9e1028F5F1D5eDE59748FFceE5532509976840E0","name":"Compound","symbol":"COMP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/COMP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x8Ee73c484A26e0A5df2Ee2a4960B789967dd0415","name":"Curve DAO Token","symbol":"CRV","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CRV/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x417Ac0e078398C154EdFadD9Ef675d30Be60Af93","name":"Curve.Fi USD Stablecoin","symbol":"crvUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/crvUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1f4446fAAAed23090f324f051C3F8c5ce5aD1c36","name":"CryoDAO","symbol":"CRYO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CRYO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x136B80f6Aa5d03456eAAE779747CC0a5F6835899","name":"Travel Deals","symbol":"CTRAVL","decimals":4,"logoURI":"https://ethereum-optimism.github.io/data/CTRAVL/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x259Fac10c5CbFEFE3E710e1D9467f70a76138d45","name":"Cartesi Token","symbol":"CTSI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CTSI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xBB22Ff867F8Ca3D5F2251B4084F6Ec86D4666E14","name":"Cryptex","symbol":"CTX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CTX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4493faE71502871245B7049580b3195bC930e661","name":"Chad USD","symbol":"CUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/CUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x91e1D2B9b0D9AfCF1F024dF09A68799fB200bc63","name":"Prime","symbol":"D2D","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/D2D/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x61431b49D43220a53B4a2583870D69F67cEd8d76","name":"Deribet DEGEN","symbol":"dbDEGEN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/dbDEGEN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x178929287b25fe1aC9e0b112CE3899312b6d0557","name":"dForce","symbol":"DF","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DF/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD7ea8433B434223b3DA2931620391Df157D1c22d","name":"Dimo","symbol":"DIMO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DIMO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xAfb89a09D82FBDE58f18Ac6437B3fC81724e4dF6","name":"The Doge NFT","symbol":"DOG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DOG/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x881Ed0FCeF78120A135eC6cC66cEf2779FE95BBA","name":"DogeGF","symbol":"DOGEGF","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DOGEGF/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4621b7A9c75199271F773Ebd9A499dbd165c3191","name":"Dola USD Stablecoin","symbol":"DOLA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DOLA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x455234AB787665a219125235fB01b22b512DfCaA","name":"DOSE","symbol":"DOSE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DOSE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x5a01BDaA010938Eb7615c63cB22cf3e94e8505df","name":"DRODEC","symbol":"DRODEC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DRODEC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x5b2124D427fAc9C80c902cbDD74b03Dd85d7d3FE","name":"Dypius","symbol":"DYP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/DYP/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x287f0D88e29a3D7AEb4d0c10BAE5B902dB69B17D","name":"Epoch","symbol":"EPOCH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/EPOCH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xACAd1d3107808577022387C07512E7828694B2a2","name":"Ethix","symbol":"ETHIX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ETHIX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x717d31A60a9e811469673429c9F8Ea24358990f1","name":"Everyworld","symbol":"EVERY","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/EVERY/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xd33c7B753ECaa85E5D5F5B5fD99Dec59f26a087E","name":"Defactor","symbol":"FACTR","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FACTR/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD08a2917653d4E460893203471f0000826fb4034","name":"FARM Reward Token","symbol":"FARM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FARM/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x076Bf099C7aaBd0BC9bC37930113428906F51d89","name":"DEFLI","symbol":"FLI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FLI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xb008BDCF9CdFf9da684a190941dC3dCa8C2Cdd44","name":"Flux","symbol":"FLUX","decimals":8,"logoURI":"https://ethereum-optimism.github.io/data/FLUX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6059d0ed9368c36941514d2864fD114a84853d5a","name":"FOAM Token","symbol":"FOAM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FOAM/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x968B2323d4b005C7D39c67D31774FE83c9943A60","name":"Ampleforth Governance","symbol":"FORTH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FORTH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2dbe0d779c7A04F7a5de83326973effE23356930","name":"ShapeShift FOX","symbol":"FOX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FOX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x90eb5f8310C6d2289Ef2475D9B8aE44589663D58","name":"FXN Token","symbol":"FXN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/FXN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x63525b8F9a78251611aDA0A05724EeeF48100fbd","name":"f(x) USD","symbol":"fxUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/fxUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2D189eABb667aA1ecFC01963a6a3A5d83960f558","name":"GALAXIS Token","symbol":"GALAXIS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GALAXIS/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1db0c569ebb4a8b57AC01833B9792F526305e062","name":"GENOME","symbol":"GENOME","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GENOME/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf0345Ba710F1354e41B808737a35A517dE5397dB","name":"GG Token","symbol":"GGTK","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GGTK/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xcD2F22236DD9Dfe2356D7C543161D4d260FD9BcB","name":"Aavegotchi GHST Token","symbol":"GHST","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GHST/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4032758A0A5dADC52ea7206fBa72aE3BBf2A44E1","name":"GigaChad","symbol":"GIGACHAD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GIGACHAD/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x09188484e1Ab980DAeF53a9755241D759C5B7d60","name":"Rigo Token","symbol":"GRG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GRG/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x321725ee44CB4bfA544CF45a5A585b925d30A58C","name":"ValleyDAO Token","symbol":"GROW","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/GROW/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3a38dde9824e18CC4C0A147824F95Bf5d608F0B3","name":"HairDAO","symbol":"HAIR","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/HAIR/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x9e5AAC1Ba1a2e6aEd6b32689DFcF62A509Ca96f3","name":"HanChain","symbol":"HAN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/HAN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3e7eF8f50246f725885102E8238CBba33F276747","name":"HANePlatform","symbol":"HANeP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/HANeP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2EDe02eC52Cbb49E81885c8fa171C3Ba8bcD5374","name":"Honey Badger","symbol":"HOBA","decimals":9,"logoURI":"https://ethereum-optimism.github.io/data/HOBA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc5102fE9359FD9a28f877a67E36B0F050d81a3CC","name":"Hop","symbol":"HOP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/HOP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x37f0c2915CeCC7e977183B8543Fc0864d03E064C","name":"HuntToken","symbol":"HUNT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/HUNT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE7798f023fC62146e8Aa1b36Da45fb70855a77Ea","name":"iFARM","symbol":"iFARM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/IFARM/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x18E692c03De43972Fe81058f322fa542Ae1A5E2c","name":"Imagine AI","symbol":"imgnAI","decimals":9,"logoURI":"https://ethereum-optimism.github.io/data/IMGNAI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x235226d2050C001FE78dED104364F9C2eB852E42","name":"Infected","symbol":"INFC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/INFC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xBCBAf311ceC8a4EAC0430193A528d9FF27ae38C1","name":"IoTeX Network","symbol":"IOTX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/IOTX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x17d2628D30F8e9E966c9Ba831c9B9b01ea8Ea75C","name":"ISKRA TOKEN","symbol":"ISK","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ISK/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xB00d803CB2367a7DA82351DCb9D213230da7B92A","name":"IYKYK","symbol":"IYKYK","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/IYKYK/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xFf9957816c813C5Ad0b9881A8990Df1E3AA2a057","name":"Geojam","symbol":"JAM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/JAM/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xde0D2ee637D3e4Fd02bc99508CA5e94dd7055766","name":"Jarvis Reward Token","symbol":"JRT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/JRT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x64cc19A52f4D631eF5BE07947CABA14aE00c52Eb","name":"Kibble","symbol":"KIBBLE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/KIBBLE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xC7DcCA0a3e69bD762C8DB257f868f76Be36c8514","name":"KiboShib","symbol":"KIBSHI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/KIBSHI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x28fe69Ff6864C1C218878BDCA01482D36B9D57b1","name":"Kyber Network Crystal v2","symbol":"KNC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/KNC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x8f019931375454Fe4EE353427eB94E2E0C9e0a8C","name":"KOMPETE Token","symbol":"KOMPETE","decimals":10,"logoURI":"https://ethereum-optimism.github.io/data/KOMPETE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xdfF3C626De2Ccd1ECf67E97abc8A74C102C86545","name":"Kromatika","symbol":"KROM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/KROM/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xfFB9CA5A2A0844535461f43429696939d5897869","name":"LIF3","symbol":"LIF3","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LIF3/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE0742c2d2Bdbb3c138Ce81Bc5589DC07865f0a49","name":"LikeButton.eth","symbol":"❤️","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LIKE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1c4076e33D1A165636f89F1A2025271e238C25d7","name":"Theranos Coin","symbol":"LIZ","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LIZ/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x977ad78482888CE57846b0A19b07Cf8d1aac6038","name":"LOCGame","symbol":"LOCG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LOCG/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x5259384690aCF240e9b0A8811bD0FFbFBDdc125C","name":"LQTY","symbol":"LQTY","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LQTY/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x0D760ee479401Bb4C40BDB7604b329FfF411b3f2","name":"LoopringCoin V2","symbol":"LRC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LRC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xB676f87A6E701f0DE8De5Ab91B56b66109766DB1","name":"BLOCKLORDS","symbol":"LRDS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LRDS/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x368181499736d0c0CC614DBB145E2EC1AC86b8c6","name":"LUSD Stablecoin","symbol":"LUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/LUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x45D9C101a3870Ca5024582fd788F4E1e8F7971c3","name":"MASQ","symbol":"MASQ","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MASQ/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x24fcFC492C1393274B6bcd568ac9e225BEc93584","name":"Heroes of Mavia","symbol":"MAVIA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MAVIA/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x8Fbd0648971d56f1f2c35Fa075Ff5Bc75fb0e39D","name":"MBS","symbol":"MBS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MBS/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc48823EC67720a04A9DFD8c7d109b2C3D6622094","name":"Metacade","symbol":"MCADE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MCADE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x93dc5Cb35627A759848c7A7F0079EA7b49E435a5","name":"Metronome2","symbol":"MET","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MET/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE8740FF403060DaE99c63A684E0DfE3eaa97e9Bc","name":"MONKE","symbol":"MONKE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MONKE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6b20e045cD881EeF5853EB6C148E1c88a20ab395","name":"dotmoovs","symbol":"MOOV","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MOOV/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x85267fBC6e1e1563bB9247336F05D9220eA99f52","name":"Empower Token","symbol":"MPWR","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MPWR/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xba7D47f471ADD16875e765edEe0858C3413A56Fd","name":"Meta","symbol":"MTA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/MTA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE009e453Ac69B0E9602BA091D5331B31B03D3039","name":"Mugloo","symbol":"MUGLOO","decimals":4,"logoURI":"https://ethereum-optimism.github.io/data/MUGLOO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3568c7a4f7545805e379c264303239781B4E9A79","name":"Cerebrum DAO Token","symbol":"NEURON","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/NEURON/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x37289326b7Bca5a776A5071b8d693C0588c5C9A6","name":"Feisty Doge","symbol":"NFD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/NFD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc2106ca72996e49bBADcB836eeC52B765977fd20","name":"NFTEarthOFT","symbol":"NFTE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/NFTE/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x13741C5dF9aB03E7Aa9Fb3Bf1f714551dD5A5F8a","name":"Noggles","symbol":"NOGS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/NOGS/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x0a93a7BE7e7e426fC046e204C44d6b03A302b631","name":"Nouns","symbol":"NOUNS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/NOUNS/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x7Cea109FC3516eD1248ae9AA67B5a352cF74075e","name":"Nova DAO","symbol":"NOVA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/NOVA/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x7002458B1DF59EccB57387bC79fFc7C29E22e6f7","name":"OriginToken","symbol":"OGN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/OGN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x54330d28ca3357F294334BDC454a032e7f353416","name":"Autonolas","symbol":"OLAS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/OLAS/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3992B27dA26848C2b19CeA6Fd25ad5568B68AB98","name":"MANTRA","symbol":"OM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/OM/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3792DBDD07e87413247DF995e692806aa13D3299","name":"OMI Token","symbol":"OMI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/OMI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x9A6d24c02eC35ad970287eE8296D4D6552a31DbE","name":"Open Ecosystem Token","symbol":"OPN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/OPN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x224114c444e3C3d936532fE08351648700570909","name":"Paladin Token","symbol":"PAL","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PAL/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xB4fDe59a779991bfB6a52253B51947828b982be3","name":"Pepe","symbol":"PEPE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PEPE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x76ff2AB6b34142421F44A68cC8Dd08f45F9Ee2F2","name":"PIP","symbol":"PIP","decimals":9,"logoURI":"https://ethereum-optimism.github.io/data/PIP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x7C34073C56285944F9A5384137186abFAe1C3bf0","name":"Plague","symbol":"PLG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PLG/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xe0231bb166D09A3f0739F7AFAd68438C0fA142a5","name":"Ponder","symbol":"PNDR","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PNDR/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xd652C5425aea2Afd5fb142e120FeCf79e18fafc3","name":"PoolTogether","symbol":"POOL","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/POOL/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x7422f030958Edf9884cE9c46a211029E03eF6da7","name":"Pepe's Dog","symbol":"POPO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/POPO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3816dD4bd44c8830c2FA020A5605bAC72FA3De7A","name":"Presearch","symbol":"PRE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PRE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xfA980cEd6895AC314E7dE34Ef1bFAE90a5AdD21b","name":"Prime","symbol":"PRIME","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PRIME/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x18dD5B087bCA9920562aFf7A0199b96B9230438b","name":"Propy","symbol":"PRO","decimals":8,"logoURI":"https://ethereum-optimism.github.io/data/PRO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x38815A4455921667d673B4cb3d48F0383eE93400","name":"pSTAKE Finance","symbol":"PSTAKE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/PSTAKE/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x197D38DC562DfB2490eC1A1d5C4CC4319d178Bb4","name":"RAC","symbol":"RAC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RAC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6AF3cb766D6cd37449bfD321D961A61B0515c1BC","name":"RAZOR","symbol":"RAZOR","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RAZOR/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x41E99e0F73a88947C52070FF67C19B7aBc171A54","name":"Redemption Finance","symbol":"RDMP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RDMP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4379c13143Eb91148fF9282CFb2f93536687A45b","name":"Reach","symbol":"$Reach","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/REACH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xB6fe221Fe9EeF5aBa221c348bA20A1Bf5e73624c","name":"Rocket Pool ETH","symbol":"rETH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/rETH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x8E5E9DF4F0EA39aE5270e79bbABFCc34203A3470","name":"Revenue Generating USD","symbol":"rgUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/rgUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf501E4c51dBd89B95de24b9D53778Ff97934cd9c","name":"DAOSquare Governance Token","symbol":"RICE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RICE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x0A4c8Ee5a092Ba333B54C142634c871768891558","name":"GenRize","symbol":"RIZE","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RIZE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1f73EAf55d696BFFA9b0EA16fa987B93b0f4d302","name":"Rocket Pool Protocol","symbol":"RPL","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RPL/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xFbB75A59193A3525a8825BeBe7D4b56899E2f7e1","name":"ResearchCoin","symbol":"RSC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/RSC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2578CDD1603A5a7Db9DF46998d2E13d0e51db527","name":"Salad","symbol":"SALD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SALD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xF3907Bc0FFF5Ff5aCf1E3dD7987005779C7bf57d","name":"Sarcophagus","symbol":"SARCO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SARCO/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2A3Ed1738D7339995914C380a02e28e8AE9Dce3b","name":"Sandclock WETH Vault v2","symbol":"scWETHv2","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/scWETHv2/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x99aC4484e8a1dbd6A185380B3A811913Ac884D87","name":"Savings Dai","symbol":"sDAI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/sDAI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1C7a460413dD4e964f96D8dFC56E7223cE88CD85","name":"Seamless","symbol":"SEAM","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SEAM/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD1917629B3E6A72E6772Aab5dBe58Eb7FA3C2F33","name":"Settled ETHXY Token","symbol":"SEXY","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SEXY/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x606B2Db8Bf642677Fe12E8F067bF55145A350cb5","name":"ShibaDoge","symbol":"ShibDoge","decimals":9,"logoURI":"https://ethereum-optimism.github.io/data/ShibDoge/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xF734eFdE0C424BA2B547b186586dE417b0954802","name":"Silo Governance Token","symbol":"Silo","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/Silo/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xd0D1e44FC9aDAEB732F73fFC2429CD1dB9cD4529","name":"Sipher Token","symbol":"SIPHER","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SIPHER/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xE7399151B688A265F347693d358821a5A8c213Ec","name":"Skillful AI","symbol":"SKAI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SKAI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2974dC646e375e83bd1c0342625b49f288987fA4","name":"Swarm Markets","symbol":"SMT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SMT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x2809ee30C77d7887e4F77394a1Ac0E49De0E397F","name":"Real Smurf Cat","symbol":"SMURFCAT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SMURFCAT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x04D1963C76EB1BEc59d0eEb249Ed86F736B82993","name":"SoftDAO","symbol":"SOFT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SOFT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xA5649B56AA4e74d8661Ee9656130072df7666784","name":"SpaceChainV2","symbol":"SPC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SPC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x8f2E6758C4D6570344bd5007DEc6301cd57590A0","name":"SPOT","symbol":"SPOT","decimals":9,"logoURI":"https://ethereum-optimism.github.io/data/SPOT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x50dA645f148798F68EF2d7dB7C1CB22A6819bb2C","name":"SPX6900","symbol":"SPX","decimals":8,"logoURI":"https://ethereum-optimism.github.io/data/SPX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x858c50C3AF1913b0E849aFDB74617388a1a5340d","name":"SubQueryToken","symbol":"SQT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SQT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x77BA5713F0AFceDB4A10FA155b5e95218A7B658E","name":"SUDO GOVERNANCE TOKEN","symbol":"SUDO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SUDO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xA9C9Dcb054421DEDc9d9006CFfe843D0a5Cd6339","name":"Sustainable Growth","symbol":"SUS","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SUS/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x7D49a065D17d6d4a55dc13649901fdBB98B2AFBA","name":"SushiToken","symbol":"SUSHI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/SUSHI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x236aa50979D5f3De3Bd1Eeb40E81137F22ab794b","name":"Base tBTC v2","symbol":"tBTC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/tBTC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4FB9B20DaFE45d91ae287f2E07B2e79709308178","name":"Tokenomy","symbol":"TEN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TEN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x5E42c17CAEab64527D9d80d506a3FE01179afa02","name":"Tetu Token","symbol":"TETU","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TETU/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf34e0cff046e154CAfCae502C7541b9E5FD8C249","name":"Optimistic Thales Token","symbol":"THALES","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/THALES/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6A38905c6EB562BAe76f1Af0D0F98F2801fb2490","name":"Current Thing","symbol":"THING","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/THING/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xb363c1235734f4C036Eb82C84C87e099C316fb48","name":"THX Network","symbol":"THX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/THX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x1287A235474E0331C0975e373bdD066444d1Bd35","name":"TAIKAI Token","symbol":"TKAI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TKAI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x78b3C724A2F663D11373C4a1978689271895256f","name":"Token Name Service","symbol":"TKN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TKN/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xf7C1CEfCf7E1dd8161e00099facD3E1Db9e528ee","name":"TOWER","symbol":"TOWER","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TOWER/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xA81a52B4dda010896cDd386C7fBdc5CDc835ba23","name":"Trace Token","symbol":"TRAC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TRAC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xF8e9E61FFB2b491f7DF29823a76009743671CD96","name":"Tellor","symbol":"TRB","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TRB/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xFEE5178BB6aD41E7D5BBC7f6145fEB27460dD1Ed","name":"The Secret Coin","symbol":"TSC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TSC/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x029a3b0532871735809A51E8653d6017eF04b6Fa","name":"TYBENG","symbol":"TYBENG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/TYBENG/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x06de990abC8E6dC2E1a9Ee10E402b3AfCeF5Cdb0","name":"U-Coin","symbol":"U","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/U/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD7eA82D19f1f59FF1aE95F1945Ee6E6d86A25B96","name":"Unlock Discount Token","symbol":"UDT","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/UDT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc3De830EA07524a0761646a6a4e4be0e114a3C83","name":"Uniswap","symbol":"UNI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/UNI/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xaC27fa800955849d6D17cC8952Ba9dD6EAA66187","name":"UnlockProtocolToken","symbol":"UP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/UP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x38547D918b9645F2D94336B6b61AEB08053E142c","name":"USC","symbol":"USC","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/USC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xEFb97aaF77993922aC4be4Da8Fbc9A2425322677","name":"Web 3 Dollar","symbol":"USD3","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/USD3/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4F604735c1cF31399C6E711D5962b2B3E0225AD3","name":"Glo Dollar","symbol":"USDGLO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/USDGLO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x490a4B510d0Ea9f835D2dF29Eb73b4FcA5071937","name":"VitaDAO Token","symbol":"VITA","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/VITA/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc8a7B50498f7D3Da97249DE165908F0f588490ED","name":"VesperToken","symbol":"VSP","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/VSP/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x0937876EFd6C4101Be68cd89ba58D5Ecf0d53A64","name":"VUSD","symbol":"VUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/VUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x0BbbEad62f7647AE8323d2cb243A0DB74B7C2b80","name":"Ambire Wallet","symbol":"WALLET","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/WALLET/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x489fe42C267fe0366B16b0c39e7AEEf977E841eF","name":"Wrapped Ampleforth","symbol":"WAMPL","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/WAMPL/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x4c94DE27c94962Dba6Ebb77924Ac54189db75EFA","name":"Wrapped eETH","symbol":"weETH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/weETH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xAc12F930318Be4F9d37f602cBF89CD33E99aa9D4","name":"WEXO","symbol":"WEXO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/WEXO/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xfB18511F1590a494360069F3640c27d55c2B5290","name":"Wild Goat Coin","symbol":"WGC","decimals":6,"logoURI":"https://ethereum-optimism.github.io/data/WGC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xDf690C65d067035364a58369C26820D3696D7799","name":"Wrapped HOGE","symbol":"wHOGE","decimals":9,"logoURI":"https://ethereum-optimism.github.io/data/wHOGE/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3d2EbA645c44BBD32A34b7c017667711eb5b173C","name":"Wrapped MistCoin","symbol":"WMC","decimals":2,"logoURI":"https://ethereum-optimism.github.io/data/WMC/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xcC2a24405480242f8EE5d9b0aE27861D801d7FFA","name":"OpenX Optimism","symbol":"wOpenX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/wOpenX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc11158c5dA9db1D553ED28f0C2BA1CbEDD42CFcb","name":"Wrapped PAW","symbol":"wPAW","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/WPAW/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3BFc2bb3B3c187ddC79e38d4A17c830E2a380b70","name":"Wrapped Pocket","symbol":"wPOKT","decimals":6,"logoURI":"https://ethereum-optimism.github.io/data/wPOKT/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xc1CBa3fCea344f92D9239c08C0568f6F2F0ee452","name":"Wrapped liquid staked Ether 2.0","symbol":"wstETH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/wstETH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD7B99ffB8B2afc6fe013a17207cbe50f223aDc94","name":"XY Oracle","symbol":"XYO","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/XYO/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xCaf1c21DE4214ab2bCDb3A55deEB0114C8EeADa1","name":"Yearn Ether","symbol":"yETH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/yETH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6F74bAf5A0F209609e5826AcBebc90d394830cf7","name":"YFX Token","symbol":"YFX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/YFX/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xaAC78d1219c08AecC8e37e03858FE885f5EF1799","name":"Yield Guild Games Token","symbol":"YGG","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/YGG/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6AEde1aec1C4062E4ED2e91F14877c04db97b3F6","name":"YokAI: Generative Monsters","symbol":"YOKAI","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/YOKAI/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xD1dB4851bcF5B41442cAA32025Ce0Afe6B8EabC2","name":"Based ZoomerCoin","symbol":"ZOOMER","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ZOOMER/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x3bB4445D30AC020a84c1b5A8A2C6248ebC9779D0","name":"0x Protocol Token","symbol":"ZRX","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ZRX/logo.png","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0xB5617AC8918CDC0e877c9Fb7e75B9dF62DBD539B","name":"Zunami Governance Token","symbol":"ZUN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ZUN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x40AeED4852Df0Ae3624E7027DEdDFb972Dbc8852","name":"Zunami ETH","symbol":"zunETH","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/zunETH/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x6ef3864876424ADE834a7C1BE8C0CF3d71208B84","name":"Zunami USD","symbol":"zunUSD","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/zunUSD/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"},{"chainId":8453,"address":"0x82c8F48AC694841360dE84d649a0d48d239B61F8","name":"ZynCoin","symbol":"ZYN","decimals":18,"logoURI":"https://ethereum-optimism.github.io/data/ZYN/logo.svg","chainType":"evm","chainIDStr":"8453","chainCode":"base-erc20"}]}
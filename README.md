# AL365TokenWithSecurity.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    // ... (önceki IERC20 interface detayları burada)
}

contract AL365TokenWithSecurity is IERC20 {
    string public constant name = "AL365";
    string public constant symbol = "AL365";
    uint8 public constant decimals = 18;
    uint256 private _totalSupply = 240000000 * (10 ** uint256(decimals));

    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    
    // Güvenlik için eklenen değişkenler
    mapping(address => uint256) private _lastTransactionTime;
    uint256 private _transactionCooldown = 60 seconds; // Her işlem arasında 1 dakika beklemek için
    uint256 private _maxTransactionAmount = 10000 * (10 ** uint256(decimals)); // Her işlemde en fazla 10,000 token

    constructor() {
        _balances[msg.sender] = _totalSupply;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }

    // ... (önceki fonksiyonlar burada)

    function transfer(address recipient, uint256 amount) external override returns (bool) {
        require(_canTransact(msg.sender, amount), "Transfer limit exceeded or cooldown not met");
        _transfer(msg.sender, recipient, amount);
        return true;
    }

    // ... (diğer IERC20 fonksiyonları burada)

    function _transfer(address sender, address recipient, uint256 amount) internal {
        require(sender != address(0), "BEP20: transfer from the zero address");
        require(recipient != address(0), "BEP20: transfer to the zero address");

        _balances[sender] = _balances[sender] - amount;
        _balances[recipient] = _balances[recipient] + amount;
        emit Transfer(sender, recipient, amount);
        
        _updateAfterTransaction(sender, amount);
    }

    // Transfer için koşulları kontrol eden yardımcı fonksiyon
    function _canTransact(address sender, uint256 amount) internal view returns (bool) {
        if(_lastTransactionTime[sender] + _transactionCooldown > block.timestamp) {
            return false; // Eğer son işlemden bu yana belirlenen süre geçmediyse
        }
        if(amount > _maxTransactionAmount) {
            return false; // Eğer işlem miktarı belirlenen maksimum miktardan fazla ise
        }
        return true;
    }

    // İşlem sonrası değişkenleri güncelleyen yardımcı fonksiyon
    function _updateAfterTransaction(address sender, uint256 amount) internal {
        _lastTransactionTime[sender] = block.timestamp;
    }
}

# ERC721A-Gas-Optimized-NFT-with-Whitelist-Merkle-Proof
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "erc721a/contracts/ERC721A.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

contract OptimizedWhitelistNFT is ERC721A, Ownable {
    bytes32 public merkleRoot;
    uint256 public maxSupply = 5000;
    uint256 public price = 0.005 ether;
    bool public saleActive;

    constructor(address initialOwner) ERC721A("OptiNFT", "ONFT") Ownable(initialOwner) {}

    function mint(uint256 quantity, bytes32[] calldata proof) external payable {
        require(saleActive, "Sale not active");
        require(totalSupply() + quantity <= maxSupply, "Exceeds supply");
        require(msg.value >= price * quantity, "Insufficient ETH");

        if (merkleRoot != bytes32(0)) {
            require(MerkleProof.verify(proof, merkleRoot, keccak256(abi.encodePacked(msg.sender))), "Not whitelisted");
        }

        _safeMint(msg.sender, quantity);
    }

    function setMerkleRoot(bytes32 _root) external onlyOwner { merkleRoot = _root; }
    function toggleSale() external onlyOwner { saleActive = !saleActive; }
    function withdraw() external onlyOwner { payable(owner()).transfer(address(this).balance); }
}

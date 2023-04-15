// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract AssetTokenization is ERC721 {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    struct Asset {
        uint256 id;
        address seller;
        string name;
        uint256 price;
        bool sold;
    }

    mapping(uint256 => Asset) public assets;

    event AssetListed(uint256 id, address seller, string name, uint256 price);
    event AssetSold(uint256 id, address buyer, uint256 price);

    constructor() ERC721("Asset Token", "AST") {}

    function listAsset(string memory _name, uint256 _price) public {
        _tokenIds.increment();
        uint256 newAssetId = _tokenIds.current();
        Asset memory newAsset = Asset(newAssetId, msg.sender, _name, _price, false);
        assets[newAssetId] = newAsset;
        emit AssetListed(newAssetId, msg.sender, _name, _price);
    }

    function buyAsset(uint256 _assetId) public payable {
        Asset storage asset = assets[_assetId];
        require(!asset.sold, "Asset has already been sold");
        require(msg.value == asset.price, "Incorrect payment amount");
        asset.sold = true;
        _safeMint(msg.sender, asset.id);
        emit AssetSold(asset.id, msg.sender, asset.price);
        payable(asset.seller).transfer(msg.value);
    }

    function getAssetDetails(uint256 _assetId) public view returns (Asset memory) {
        return assets[_assetId];
    }

    function tokenURI(uint256 _tokenId) public view override returns (string memory) {
        require(_exists(_tokenId), "Token does not exist");
        return string(abi.encodePacked("https://asset.com/assets/", uint2str(_tokenId)));
    }

    function uint2str(uint256 _i) internal pure returns (string memory) {
        if (_i == 0) {
            return "0";
        }
        uint256 j = _i;
        uint256 length;
        while (j != 0) {
            length++;
            j /= 10;
        }
        bytes memory bstr = new bytes(length);
        uint256 k = length;
        while (_i != 0) {
            k = k-1;
            uint8 temp = (48 + uint8(_i - _i / 10 * 10));
            bytes1 b1 = bytes1(temp);
            bstr[k] = b1;
            _i /= 10;
        }
        return string(bstr);
    }
}

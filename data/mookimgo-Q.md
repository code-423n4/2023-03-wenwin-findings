#1. Ticket should add a custom tokenURI for displaying ticketsInfo

Ticket.sol does not has custom tokenURI function, which leads to incorrect NFT image display, makes it hard for players to buy tickets on Opensea or other NFT marketplaces.

If tokenURI do not correctly display enough information to user, attacker can trick victim to buy his claimed Ticket.

Suggestion: Add a tokenURI function to render `drawId`, `combination`, and the most important `claimed`. This can be done in solidity by concatenating svg lines and use base64 encode.

```
//copy from https://stackoverflow.com/questions/47129173/how-to-convert-uint-to-string-in-solidity
function uint2str(uint _i) internal pure returns (string memory _uintAsString) {
        if (_i == 0) {
            return "0";
        }
        uint j = _i;
        uint len;
        while (j != 0) {
            len++;
            j /= 10;
        }
        bytes memory bstr = new bytes(len);
        uint k = len;
        while (_i != 0) {
            k = k-1;
            uint8 temp = (48 + uint8(_i - _i / 10 * 10));
            bytes1 b1 = bytes1(temp);
            bstr[k] = b1;
            _i /= 10;
        }
        return string(bstr);
    }

//inspired by Loot https://etherscan.io/address/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7#code
function tokenURI(uint256 tokenId) override public view returns (string memory) {
    ITicket.TicketInfo info = ticketsInfo[tokenId];
    string[5] memory parts;
    parts[0]='<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350"><style>.base { fill: white; font-family: serif; font-size: 14px; }</style><rect width="100%" height="100%" fill="black" /><text x="10" y="20" class="base">Wenwin Draw Ticket</text><text x="10" y="60" class="base">drawId: ';
    parts[1]=uint2str(uint256(info.drawId));
    parts[2]='</text><text x="10" y="80" class="base">Combination: ';
    parts[3]=uint2str(uint256(info.combination));
    parts[4]='</text><text x="10" y="100" class="base">Status: ';
    if(info.claimed){
        parts[5]='<font color=red>CLAIMED</font>';
    } else {
        parts[5]='Not claimed';
    }
    string memory output = string(abi.encodePacked(parts[0], parts[1], parts[2], parts[3], parts[4], parts[5]);
        
    string memory json = Base64.encode(bytes(string(abi.encodePacked('{"name": "Wenwin Draw Ticket #', uint2str(tokenId), '", "description": "NFT description HERE", "image": "data:image/svg+xml;base64,', Base64.encode(bytes(output)), '"}'))));
    output = string(abi.encodePacked('data:application/json;base64,', json));
    return output;
}
```


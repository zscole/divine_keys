·▄▄▄▄  ▪   ▌ ▐·▪   ▐ ▄ ▄▄▄ .    ▄ •▄ ▄▄▄ . ▄· ▄▌.▄▄ · 
██▪ ██ ██ ▪█·█▌██ •█▌▐█▀▄.▀·    █▌▄▌▪▀▄.▀·▐█▪██▌▐█ ▀. 
▐█· ▐█▌▐█·▐█▐█•▐█·▐█▐▐▌▐▀▀▪▄    ▐▀▀▄·▐▀▀▪▄▐█▌▐█▪▄▀▀▀█▄
██. ██ ▐█▌ ███ ▐█▌██▐█▌▐█▄▄▌    ▐█.█▌▐█▄▄▌ ▐█▀·.▐█▄▪▐█
▀▀▀▀▀• ▀▀▀. ▀  ▀▀▀▀▀ █▪ ▀▀▀     ·▀  ▀ ▀▀▀   ▀ •  ▀▀▀▀ 

----------------------------------------------------------
# DIVINE KEYS: DECENTRALIZED DUNGEONS
----------------------------------------------------------
  
# OVERVIEW
Divine Keys is a fully on-chain decentralized dungeon protocol that presents an immersive and text-based game in which players create and control characters to explore and aventure in a virtual world.

Within the world of Divine Keys, players can interact with each other in a  realm, where they can engage in combat (either cooperatively or via PVP), complete quests, and build their characters. Players can communicate with each other through text-based chat, and form factions or clans within the game.

As a fully text-based game, Divine Keys can be played using a simple CLI based client or in-browser. 

# ARCHITECTURE
## Game Framework
The game framework smart contract that defines logic that for the basic rules of the game, including character creation, and game world structure.
The contract includes a list of registered players, character information, and in-game assets (e.g., items, NPCs, locations).

## Character Creation
Users are able to customize their chracter's `name`, `race`, and `class`. The remaining stats are randomly generated upon mint. These stats cannot be changed once the character NFT has been minted. Instead, these stats can be modified, or boosted, in combination with unique items that can be aquired in-game by completing quests, looting fallen opponents in PVP matches, found randomly in dungeons, or purchased within the marketplace. 

```
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract EtherealQuestFramework is ERC721, ReentrancyGuard {
    using Strings for uint256;
    using Counters for Counters.Counter;

    struct Character {
        uint256 characterID;
        string name;
        string race;
        string class;
        uint256 strength;
        uint256 dexterity;
        uint256 intelligence;
    }

    uint256 private characterCount = 0;

    mapping(uint256 => Character) public characters;
    Counters.Counter private tokenId;

    constructor() ERC721("EtherealQuestCharacters", "EQC") {}

    function createCharacter(
        string memory _name,
        string memory _race,
        string memory _class
    ) public nonReentrant {
        tokenId.increment();
        uint256 newItemId = tokenId.current();

        uint256 strength = _randomAttribute(newItemId, "STR");
        uint256 dexterity = _randomAttribute(newItemId, "DEX");
        uint256 intelligence = _randomAttribute(newItemId, "INT");

        characters[newItemId] = Character(newItemId, _name, _race, _class, strength, dexterity, intelligence);
        _mint(msg.sender, newItemId);
    }

    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
        require(_exists(tokenId), "Token does not exist");

        Character memory character = characters[tokenId];

        string memory svg = generateSVG(character);

        string memory json = Base64.encode(
            bytes(
                string(
                    abi.encodePacked(
                        '{"name": "', character.name, '", "description": "An Ethereal Quest character", "image": "data:image/svg+xml;base64,', Base64.encode(bytes(svg)), '"}'
                    )
                )
            )
        );

        return string(abi.encodePacked("data:application/json;base64,", json));
    }

    function generateSVG(Character memory character) internal pure returns (string memory) {
        string memory svg = string(
            abi.encodePacked(
                '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 300 300">',
                // Add SVG elements to represent the character's race, class, attributes, etc.
                '</svg>'
            )
        );

        return svg;
    }

    function _randomAttribute(uint256 tokenId, string memory salt) internal view returns (uint256) {
        uint256 randomNumber = uint256(keccak256(abi.encodePacked(tokenId, salt, block.timestamp))) % 20;
        return randomNumber + 1;
    }
}

library Base64 {
    bytes internal constant TABLE = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

    // ... (Base64 encoding and decoding functions)
```
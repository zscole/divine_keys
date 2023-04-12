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

contract DivineKeysFramework is ERC721, ReentrancyGuard {
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

    constructor() ERC721("DivineKeysCharacters", "DKC") {}

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
                        '{"name": "', character.name, '", "description": "A Divine Keys character", "image": "data:image/svg+xml;base64,', Base64.encode(bytes(svg)), '"}'
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
                '<rect width="100%" height="100%" fill="white"/>',
                '<text x="10" y="20" font-family="Verdana" font-size="14">Name: ', character.name, '</text>',
                '<text x="10" y="40" font-family="Verdana" font-size="14">Race: ', character.race, '</text>',
                '<text x="10" y="60" font-family="Verdana" font-size="14">Class: ', character.class, '</text>',
                '<text x="10" y="80" font-family="Verdana" font-size="14">Strength: ', character.strength.toString(), '</text>',
                '<text x="10" y="100" font-family="Verdana" font-size="14">Dexterity: ', character.dexterity.toString(), '</text>',
                '<text x="10" y="120" font-family="Verdana" font-size="14">Intelligence: ', character.intelligence.toString(), '</text>',
                '</svg>'
            )
        );

        return svg;
    }

function _randomAttribute(uint256 tokenId, string memory salt) internal view returns (uint256) {
    uint256 randomValue = uint256(keccak256(abi.encodePacked(block.timestamp, tokenId, salt)));
    return (randomValue % 20) + 1;
}
```

## DUNGEONS
Each dungeon is a traversable graph, however, we represent this graph as a matrix since it's in a smart contract. Users explore this dungeon in the same way they would traverse this graph.
```
pragma solidity ^0.8.0;

contract Dungeon {
    uint8 public constant MATRIX_SIZE = 8;
    uint256[MATRIX_SIZE][MATRIX_SIZE] public matrix;

    function setMatrixValue(uint8 x, uint8 y, uint256 value) public {
        require(x < MATRIX_SIZE, "X-coordinate out of bounds");
        require(y < MATRIX_SIZE, "Y-coordinate out of bounds");
        matrix[x][y] = value;
    }

    function getMatrixValue(uint8 x, uint8 y) public view returns (uint256) {
        require(x < MATRIX_SIZE, "X-coordinate out of bounds");
        require(y < MATRIX_SIZE, "Y-coordinate out of bounds");
        return matrix[x][y];
    }
}
```

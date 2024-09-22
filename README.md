---

# Unity Data Persistence System

This project showcases a custom data persistence system for Unity, designed to efficiently save and load game data across multiple game sessions. It includes features such as encryption, auto-save, profile management, and a modular interface for saving various types of in-game data (e.g., player stats, collectibles).

## Features

- **Custom Data Persistence:** Implements a flexible system for managing game data using C# and manual file I/O operations, with optional XOR encryption.
- **Encryption Support:** XOR encryption for secure game data storage, ensuring sensitive information remains protected.
- **Auto-Save:** Automatically saves game data at configurable intervals, ensuring game progress is not lost.
- **Profile Management:** Handles multiple player profiles with seamless switching, each having its own independent game data.
- **Modular Data Saving:** Easily extendable through the `IDataPersistence` interface, allowing different in-game objects to save/load their specific data.
- **Backup and Rollback System:** Automatically creates backup files and attempts rollback if data corruption occurs during save/load operations.

## Code Overview

### 1. **`DataPersistenceManager.cs`**
   - The core class responsible for managing the save and load cycles of game data.
   - Handles initialization, auto-save, and profile switching.
   - Uses a list of objects that implement the `IDataPersistence` interface to save and load their specific data.
   - Provides a **New Game**, **Load Game**, and **Save Game** API for easy integration.

### Key Methods:
   - **`LoadGame()`**: Loads game data for the current profile and triggers all associated objects to load their respective data.
   - **`SaveGame()`**: Triggers all game objects to save their data and writes the serialized game data to a file.
   - **`ChangeSelectedProfileId()`**: Allows changing between multiple player profiles and updates the game state accordingly.

### 2. **`FileDataHandler.cs`**
   - Manages file input/output operations, including reading from and writing to the game data file.
   - Supports optional encryption and decryption during the save/load processes.
   - Contains a backup mechanism for safeguarding against potential file corruption.

### Key Methods:
   - **`Load()`**: Loads game data from a file, with an option to restore from a backup in case of errors.
   - **`Save()`**: Writes game data to a file and creates a backup after verifying the save.
   - **`EncryptDecrypt()`**: A simple XOR encryption algorithm used for encoding game data.

### 3. **`SerializableDictionary<TKey, TValue>.cs`**
   - A custom serializable dictionary class used to store collections of data (e.g., collected items) in Unity.
   - Unity’s native `Dictionary` type is not serializable, so this class solves that limitation.

### 4. **`GameData.cs`**
   - A container class that stores all relevant data for the game state, such as player health, position, collected items, and attributes.
   - Includes a method for calculating percentage completion based on collected items.

### 5. **`IDataPersistence.cs` (Interface)**
   - A modular interface that game objects can implement to save and load their specific data.
   - Ensures a standardized approach to data persistence across different game objects (e.g., coins, power-ups, stats).

### Methods:
   - **`LoadData(GameData data)`**: Loads specific data for an object.
   - **`SaveData(GameData data)`**: Saves specific data for an object.

### 6. **`Coin.cs`**
   - A sample implementation of `IDataPersistence` for managing collectible items.
   - Each coin has a unique ID generated via a GUID.
   - Handles coin collection and updates the `coinsCollected` dictionary in `GameData`.

### Key Methods:
   - **`LoadData(GameData data)`**: Loads the state of the coin (whether it was collected or not) based on the saved data.
   - **`SaveData(GameData data)`**: Saves the coin’s collection status in the game data.
   - **`OnTriggerEnter2D()`**: Detects when a player collects the coin and updates the game state.

## Getting Started

### Prerequisites
- Unity 2020.3+ (or later)
- Basic knowledge of C# scripting in Unity.

### Installation
1. Clone or download the repository:
   ```bash
   git clone https://github.com/Chris91ss/DataPersistence-System
   ```
2. Open the project in Unity.
3. Attach the `DataPersistenceManager` to a GameObject in your scene.
4. Implement the `IDataPersistence` interface on any object that requires saving and loading data.

### Usage
- **New Game**: Call `DataPersistenceManager.instance.NewGame()` to initialize a fresh game session.
- **Save Game**: Call `DataPersistenceManager.instance.SaveGame()` to save the current game state.
- **Load Game**: Call `DataPersistenceManager.instance.LoadGame()` to load a previously saved game.
- **Switch Profile**: Use `DataPersistenceManager.instance.ChangeSelectedProfileId()` to switch between different player profiles.

## Auto-Save Configuration

By default, the game auto-saves every 60 seconds. To adjust this:
- Go to the `DataPersistenceManager` script and change the `autoSaveTimeSeconds` field to the desired value.

## Data Encryption

To enable or disable encryption:
- In the `FileDataHandler.cs` script, set the `useEncryption` flag to `true` or `false`.

The system uses a simple XOR (exclusive OR) encryption algorithm to encrypt and decrypt the game data. This method provides basic protection by obscuring the game data stored in the file, preventing casual access or tampering. However, it's not intended to be highly secure encryption.

### How XOR Encryption Works:
The XOR algorithm works by applying the XOR operation between each character of the data string and a predefined code word. This process is symmetric, meaning the same operation is used for both encryption and decryption.

Here's a step-by-step breakdown:
1. Each character in the string representing the game data is XORed with a corresponding character from the **encryption code word**. 
2. If the data string is longer than the code word, the code word is looped over to match the length of the data.
3. The result is an encrypted string where the original data is obscured.
4. To decrypt the data, the same XOR operation is applied again with the same code word, restoring the original text.

### Example:
Given the data string `playerHealth:100` and a code word `word`, the encryption process would look like this:

- First, the string is XORed with the code word:
  - XOR each character in `playerHealth:100` with the characters of `word`. If the string is longer than the code word, loop through the code word until all characters are XORed.
  
  Encrypted result: `œÙùøÛæ¹éÅÿ½þ` (an unreadable string)

- To decrypt:
  - Apply the XOR operation again using the same code word `word`, which restores the original data: `playerHealth:100`.

This is implemented in the method `EncryptDecrypt()` in `FileDataHandler.cs`.

### Code Example:
```csharp
private string EncryptDecrypt(string data)
{
    string modifiedData = "";
    for (int i = 0; i < data.Length; i++) 
    {
        modifiedData += (char)(data[i] ^ encryptionCodeWord[i % encryptionCodeWord.Length]);
    }
    return modifiedData;
}
```

### Limitations of XOR Encryption:
- **Basic Security:** XOR encryption provides a low level of security. If someone knows the code word or the encryption pattern, they can easily decrypt the data.
- **Vulnerability to Repetition Attacks:** Since the code word is repeated for longer data strings, it can become vulnerable to frequency analysis attacks or pattern recognition.

### Potential Future Improvements:
To improve security in a production environment, you could implement more advanced encryption algorithms, such as:
- **AES (Advanced Encryption Standard):** A symmetric encryption algorithm widely used for secure data encryption. It provides a significantly stronger level of protection than XOR.
- **RSA (Rivest–Shamir–Adleman):** A public-key cryptosystem that can be used for secure key exchange or encrypting small amounts of sensitive data (such as a player’s profile information).

Incorporating these algorithms would ensure that game data remains secure, even against more sophisticated attacks.

---

## Profile Management

This system supports multiple profiles. Each profile has its own independent game data. You can switch profiles dynamically by calling the `ChangeSelectedProfileId()` method and passing the new profile ID.

## Future Enhancements

- Support for more advanced data structures in `GameData`.
- Improved encryption using modern cryptographic algorithms.
- GUI for managing player profiles and saved game data.
- Enhanced backup and rollback system with version control.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

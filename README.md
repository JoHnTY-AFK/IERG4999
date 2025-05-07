# Secure Group Messaging with Cryptographic Watermarking

## Overview
This project, developed as part of the IERG4999I Final Year Project II at The Chinese University of Hong Kong (April 2025), implements a secure group messaging system with cryptographic watermarking. It integrates TreeKEM-based group key management, AES-CBC encryption, and DCT/LSB watermarking to ensure confidentiality, forward secrecy, and data authenticity. The system includes a command-line tool for encryption/decryption and a Telegram bot for secure photo sharing in group chats.

### Key Features
- **Group Key Management**: Uses TreeKEM (X25519 key exchange, HKDF) for scalable key derivation and updates, ensuring forward secrecy.
- **Encryption**: Employs AES-256-CBC for text and photo encryption with chunked processing and CRC32 checksums.
- **Watermarking**: Embeds tamper-evident markers using DCT (for images) and LSB (for metadata), ensuring authenticity and traceability.
- **Telegram Bot**: Facilitates real-time photo sharing with encrypted metadata in group chats.
- **Scalability**: Supports groups up to 100 members with efficient key updates (5-123 ms) and stable memory usage (50-60 MB).
- **Robustness**: Defends against cut attacks (cropping) with DCT's frequency-domain embedding and LSB's redundant metadata.

## Project Structure
```
PROJECT
├── output
│   ├── decrypted
│   │   └── decrypted_photo.png
│   ├── encrypted
│   │   ├── derived_key.bin
│   │   ├── encrypted_photo.bin
│   │   └── private_key.bin
│   └── src
│       ├── bot
│       │   ├── __init__.py
│       │   ├── handlers.py
│       │   └── main.py
│       ├── data
│       │   ├── photos
│       │   ├── watermarks
│       │   ├── output
│       │   │   ├── decrypted
│       │   │   ├── encrypted
│       │   │   ├── extracted
│       │   │   ├── keys
│       │   │   ├── original
│       │   │   └── temp
│       │   └── utils
│       │       ├── __init__.py
│       │       ├── crypto.py
│       │       ├── treekem.py
│       │       └── watermark.py
│       ├── __init__.py
│       ├── decrypt.py
│       └── encrypt.py
├── .env
├── requirements.txt
├── structure.txt
└── README.md
```

### Directory Details
- **output/**: Stores encrypted/decrypted files, keys, and extracted watermarks.
- **src/bot/**: Contains Telegram bot logic (`main.py`, `handlers.py`).
- **src/data/**: Holds input photos (`photos/`), watermark images (`watermarks/`), and temporary outputs.
- **src/utils/**: Utility modules for encryption (`crypto.py`), key management (`treekem.py`), and watermarking (`watermark.py`).
- **src/encrypt.py**, **decrypt.py**: Command-line tools for encryption/decryption.
- **.env**: Stores sensitive data (e.g., `TELEGRAM_BOT_TOKEN`).
- **requirements.txt**: Lists dependencies (e.g., `cryptography`, `opencv-python`, `Pillow`).

## System Components
### 1. Group Key Management (treekem.py)
- Implements TreeKEM for scalable key management.
- Uses X25519 key exchange and HKDF to derive group keys.
- Supports dynamic membership (add/remove members) with logarithmic key updates (15-30 ms).
- Ensures forward secrecy: removed members cannot decrypt new content.

### 2. Encryption/Decryption (crypto.py, encrypt.py, decrypt.py)
- **AES-256-CBC**: Encrypts text and photos with 256-bit keys, 16-byte IV, and CBC mode.
- **Chunked Processing**: Splits large data into 1024-byte chunks, each with CRC32 checksums.
- **Performance**: 
  - Text (100 chars): ~3 ms (encrypt/decrypt).
  - Photo (786 KB): ~45 ms (encrypt), ~41 ms (decrypt).
- **Error Recovery**: Handles corrupted data with partial recovery mechanisms.

### 3. Watermarking (watermark.py)
- **DCT Watermarking**: Embeds visual markers in the frequency domain (alpha=0.05).
  - Embedding: 320 ms, Extraction: 310 ms.
  - Quality: PSNR ≈ 40 dB, SSIM ≈ 0.87.
  - Robust to 90% JPEG compression and 50% cropping (85% extraction success).
- **LSB Watermarking**: Embeds encrypted metadata (e.g., group ID, timestamp).
  - Embedding: 150 ms, Extraction: 145 ms.
  - Quality: PSNR > 50 dB.
  - 75% bit accuracy after 50% cropping due to redundant embedding.
- **Cut Attack Defense**: DCT's frequency-domain and LSB's redundancy ensure watermark recovery.

### 4. Telegram Bot (main.py, handlers.py)
- **Commands**:
  - `/start`: Introduces the bot.
  - `/share`: Initiates photo sharing in a group.
  - `/view_{user_id}`: Requests a shared photo.
- **Functionality**:
  - Users upload photos privately, which are watermarked (DCT+LSB) and shared in group chats.
  - Embeds metadata (group ID, timestamp, member list) via LSB.
  - Processing Time: 500-510 ms (upload to delivery).
- **Security**: Integrates TreeKEM and AES for secure sharing.

## Setup and Usage
### Prerequisites
- Python 3.9+
- Install dependencies:
  ```
  pip install -r requirements.txt
  ```
  Dependencies include:
  - `cryptography==41.0.7`
  - `opencv-python==4.9.0`
  - `Pillow==10.2.0`
  - `numpy==1.25.4`
  - `python-telegram-bot==21.0.1`
  - `python-dotenv==1.0.1`

### Configuration
1. Create a `.env` file in the project root:
   ```
   TELEGRAM_BOT_TOKEN=your_bot_token_here
   ```
   Obtain the token by creating a bot via Telegram's BotFather.

2. Ensure input directories exist:
   - `data/photos/`: For input photos (e.g., `testphoto.png`).
   - `data/watermarks/`: For watermark images (e.g., `watermark.png`).

### Running the Command-Line Tool
- **Encrypt**:
  ```
  python src/encrypt.py
  ```
  - Choose `text` or `photo`.
  - Specify group size, input data, watermark options, and derived key filename.
  - Outputs: `output/encrypted/` (keys, encrypted data), `output/decrypted/` (watermarked photos).

- **Decrypt**:
  ```
  python src/decrypt.py
  ```
  - Choose `text` or `photo`.
  - Provide derived key filename and photo paths.
  - Outputs: Decrypted content and extracted watermarks in `output/extracted/`.

### Running the Telegram Bot
1. Start the bot:
   ```
   python src/main.py
   ```
2. In a Telegram group:
   - Use `/share` to upload a photo privately.
   - Use `/view_{user_id}` to request the photo.
3. Outputs:
   - Watermarked photo delivered with caption.
   - Keys and files stored in `output/encrypted/` and `output/decrypted/`.

## Experimental Results
- **TreeKEM Key Management**:
  - Key generation: 5.2 ms (2 members) to 123.5 ms (100 members).
  - Key updates: 15-30 ms, logarithmic scaling.
- **AES Encryption/Decryption**:
  - Text: 3.1 ms (encrypt), 2.8 ms (decrypt).
  - Photo (786 KB): 45.3 ms (encrypt), 40.7 ms (decrypt).
- **Watermarking**:
  - DCT: PSNR ≈ 40 dB, robust to 50% cropping (85% success).
  - LSB: PSNR > 50 dB, 75% bit accuracy after 50% cropping.
- **Telegram Bot**: 500-510 ms for photo sharing, 100% metadata accuracy.
- **Scalability**: Stable memory usage (50-60 MB) for up to 100 members.
- **Forward Secrecy**: Verified—removed members cannot decrypt new content.

## Future Directions
- Optimize TreeKEM with parallel key derivation for large groups.
- Integrate post-quantum cryptography (e.g., Kyber) in `treekem.py`.
- Enhance LSB watermarking with error-correcting codes (e.g., Reed-Solomon).
- Optimize DCT watermarking for faster embedding in real-time use.
- Add password-based encryption for user keys and support video sharing in the Telegram bot.

## References
- TreeKEM: [IETF Draft](https://datatracker.ietf.org/doc/draft-barnes-treekem/)
- Watermarking: Cox, I., et al., *Digital Watermarking and Steganography*, Morgan Kaufmann, 2007.
- Full references in the project report (`project.pdf`).

## Authors
- Tsoi, Ming Hon (1155175123)
- Zhang, Yi Yao (1155174982)
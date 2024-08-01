# Tic Tac Toe Application

## วิธีการติดตั้งและรันโปรแกรม

### ขั้นตอนการติดตั้ง

1. **ติดตั้ง Flutter SDK**:
    ดาวน์โหลดและติดตั้ง [Flutter SDK](https://flutter.dev/docs/get-started/install) หากยังไม่ได้ติดตั้ง

2. **ติดตั้ง Dependencies**:
    รันคำสั่งต่อไปนี้เพื่อทำการติดตั้ง dependencies ที่จำเป็นสำหรับโปรเจค:
    ```bash
    flutter pub get
    ```

### ขั้นตอนการรันโปรแกรม

1. **รันแอปพลิเคชันบน Emulator หรือ Device จริง**:
    ```bash
    flutter run
    ```

## การออกแบบโปรแกรมและ Algorithm ที่ใช้

### การออกแบบโปรแกรมและโครงสร้างไฟล์ของโปรเจค

  * main.dart: ไฟล์หลักที่ใช้ในการรันแอปพลิเคชัน และแสดงผลหน้าจอเริ่มต้น StartGame
    
  * models/game_model.dart: โมเดลที่ใช้ในการจัดการข้อมูลเกม เช่น บอร์ด ผู้เล่นปัจจุบัน และการเคลื่อนไหวของผู้เล่น
    
  * views: โฟลเดอร์ที่เก็บหน้า UI ต่างๆ ของแอปพลิเคชัน
    
    * game_board.dart: หน้า UI สำหรับแสดงบอร์ดเกมและการเล่นเกม
      
    * history_view.dart: หน้า UI สำหรับแสดงประวัติการเล่นเกมที่ผ่านมา
      
    * replay_view.dart: หน้า UI สำหรับเล่นซ้ำเกมที่บันทึกไว้
      
    * start_game.dart: หน้า UI สำหรับเริ่มต้นเกมใหม่
      
  * controllers: โฟลเดอร์ที่เก็บการควบคุมและการจัดการข้อมูลต่างๆ ของแอปพลิเคชัน
    
    * game_controller.dart: ควบคุมการทำงานของเกม เช่น การทำการเคลื่อนไหว การตรวจสอบผู้ชนะ และการบันทึกข้อมูลเกม
      
    * database_controller.dart: จัดการการเชื่อมต่อและการบันทึกข้อมูลลงในฐานข้อมูล SQLite
      
  * utils/dialog_utils.dart: ไฟล์ที่เก็บฟังก์ชันช่วยในการแสดง Dialog ต่างๆ

## Algorithm ที่ใช้
  การทำการเคลื่อนไหวของผู้เล่น:
```dart
    void makeMove(int index) {
      if (board[index] == '') {
        board[index] = currentPlayer;
        moves.add({'player': currentPlayer, 'position': index});
        currentPlayer = currentPlayer == 'X' ? 'O' : 'X';
      }
    }
```
  การตรวจสอบผู้ชนะ:
```dart
bool checkWinner() {
  // Check rows
  for (int i = 0; i < boardSize; i++) {
    if (checkLine(i * boardSize, 1, boardSize)) return true;
  }

  // Check columns
  for (int i = 0; i < boardSize; i++) {
    if (checkLine(i, boardSize, boardSize)) return true;
  }

  // Check diagonals
  if (checkLine(0, boardSize + 1, boardSize)) return true;
  if (checkLine(boardSize - 1, boardSize - 1, boardSize)) return true;

  return false;
}

bool checkLine(int start, int step, int count) {
  final String first = board[start];
  if (first == '') return false;
  for (int i = 1; i < count; i++) {
    if (board[start + i * step] != first) return false;
  }
  return true;
}
```
  การบันทึกข้อมูลเกมลงฐานข้อมูล:
```dart
Future<void> saveGames(List<String> board, String currentPlayer,
    List<Map<String, dynamic>> moves, int boardSize) async {
  final db = await database;
  try {
    String gameData = jsonEncode({
      'board': board,
      'currentPlayer': currentPlayer,
      'moves': moves,
    });
    await db.insert('games', {
      'game_data': gameData,
      'timestamp': DateTime.now().toIso8601String(),
      'board_size': boardSize,
    });
    print('Saving game data: $gameData at ${DateTime.now().toIso8601String()}, Board size: $boardSize');
  } catch (e) {
    print('Error saving game: $e');
  }
}
```
  การดึงข้อมูลประวัติการเล่นเกม:
```dart
Future<List<Map<String, dynamic>>> getGameHistory() async {
  final db = await database;
  try {
    return await db.query('games', orderBy: 'timestamp DESC');
  } catch (e) {
    print('Error retrieving game history: $e');
    return [];
  }
}
```

## การใช้งาน
* เปิดแอปพลิเคชันเพื่อเริ่มต้นเกมใหม่ โดยสามารถเลือกขนาดของบอร์ดได้ตั้งแต่ 3x3 ถึง 10x10
  
* เล่นเกมบนหน้าจอ GameBoard โดยการแตะที่ตำแหน่งที่ต้องการทำการเคลื่อนไหว
  
* เมื่อเกมจบลง จะแสดงผลผู้ชนะหรือผลเสมอ และบันทึกข้อมูลเกมลงในฐานข้อมูล
  
* สามารถดูประวัติการเล่นเกมที่ผ่านมาได้จากหน้าจอ HistoryView
  
* สามารถเล่นซ้ำเกมที่บันทึกไว้ได้จากหน้าจอ ReplayView

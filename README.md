# Maze Car

# MLGame


* 遊戲版本：`1.1`

## 更新
增加6 X 6迷宮地圖
增加控制時間的參數

## 遊戲說明

想要訓練屬於自己的迷宮自走車嗎？

想要跟朋友來場刺激的競賽嗎？

帶上你的自走車往終點衝刺吧！

### 遊戲玩法

此遊戲為迷宮自走車模擬遊戲，遊戲過程中玩家需要控制一台配備有前、中、後三個超聲波感測器的車子，目標是以最快的速度走出迷宮。除了使用鍵盤，也可以透過程式AI操作車子。

- 鍵盤操作模式：玩家使用上下左右鍵控制車子。按上鍵車子會以固定速度前進，按下鍵則後退，都沒有按則停在原地。左右鍵控制車子原地左右旋轉。
- AI操作模式：程式接收三個超聲波感測的數值後，回傳左右輪的馬力以控制車子前進。

### 遊戲規則

遊戲最多可以六個人同時進行，有普通模式和挑戰模式。

🚗普通模式：抵達迷宮終點，遊戲將記錄不同玩家完成迷宮所花費的時間。
🚨挑戰模式：玩家在闖關過程中除了計時，還需要盡可能不讓車體碰到牆壁，當累計碰到牆壁三次將視為挑戰失敗，車子只能停在原地直到遊戲結束。

![](https://i.imgur.com/ymZZMyO.png)

## 執行
* 直接執行 預設是單人遊戲
`python main.py`
    * 車子前進、後退、左轉、右轉：1P - `UP`、`DOWN`、`LEFT`、`RIGHT`，2P - `W`、`S`、`A`、`D`
    

* 搭配[MLGame](https://github.com/LanKuDot/MLGame)執行，請將遊戲放在MLGame/games資料夾中，遊戲資料夾需命名為**Maze_Car**
    * 手動模式：
`python MLGame.py -m Maze_Car <the number of user> [game_mode] <map> [sound]`
    * 機器學習模式：
`python MLGame.py -i ml_play_template.py Maze_Car <the number of user> [game_mode] <map> [sound]`

### 遊戲參數

* `the number of user`：指定遊戲玩家人數，最少需一名玩家。單機手動模式最多兩名(鍵盤位置不足)，機器學習模式至多六名。
* `game_mode`：遊戲模式，目前只有迷宮模式，預設為"MAZE"。
* `map`：選擇不同的迷宮，目前提供2種迷宮地圖，並且會隨時增加，迷宮編號從1開始，預設為1號地圖。
* `time`：控制遊戲時間，單位為秒，時間到了之後即使有玩家還沒走出迷宮，遊戲仍然會結束。
* `sound`：音效設定，可選擇"on"或"off"，預設為"off"

## 詳細遊戲資料

### 座標系

使用 pygame 的座標系統，原點在遊戲區域的左上角，x 正方向為向右，y 正方向為向下。遊戲物件的座標皆為物件的左上角。


### 遊戲物件
遊戲中的單位為公分(cm)，亦為超聲波感測器回傳值的單位。

#### 車子

* 10 \* 10 公分大小的正方形車體

#### 格子

* 30 \* 30 公分的正方形

## 撰寫玩遊戲的程式

程式範例在 [`ml/ml_play_template.py`](https://github.com/yen900611/RacingCar/blob/master/ml/ml_play_template.py)。


### 初始化參數
```python=2
def __init__(self, player):
    self.r_sensor_value = 0
    self.l_sensor_value = 0
    self.f_sensor_value = 0
    self.control_list = [{"left_PWM" : 0, "right_PWM" : 0}]
```
`"player"`: 字串。其值只會是 `"player1"` 、 `"player2"` 、 `"player3"` 、 `"player4"` 、`"player5"` 、 `"player6"` ，代表這個程式被哪一台車使用。


### 遊戲場景資訊

由遊戲端發送的字典物件，同時也是存到紀錄檔的物件。
```python=17
   def update(self, scene_info: dict):
        """
        Generate the command according to the received scene information
        """
        self.r_sensor_value = scene_info["R_sensor"]
        self.l_sensor_value = scene_info["L_sensor"]
        self.f_sensor_value = scene_info["F_sensor"]
        self.control_list[0]["left_PWM"] += 50
        self.control_list[0]["right_PWM"] += 50

        return self.control_list

```
以下是該字典物件的鍵值對應：

* `"frame"`：整數。紀錄的是第幾影格的場景資訊
* `status`：遊戲狀態
* `"L_sensor"`：玩家自己車子左邊超聲波感測器的值，資料型態為數值
* `"F_sensor"`：玩家自己車子前面超聲波感測器的值，資料型態為數值
* `"R_sensor"`：玩家自己車子右邊超聲波感測器的值，資料型態為數值

#### 遊戲指令

傳給遊戲端用來控制自走車的指令。

玩家透過字典`{"left_PWM" : 0, "right_PWM" : 0}`回傳左右輪的馬力，範圍為-255~255，並將此字典放入清單中回傳。
例如：`[{"left_PWM" : 0, "right_PWM" : 0}]`

## 機器學習模式的玩家程式

自走車可以多人遊戲，所以在啟動機器學習模式時，需要利用 `-i <script_for_1P> -i <script_for_2P> -i <script_for_3P> -i <script_for_4P>` 指定最多六個不同的玩家程式。
* For example
`python MLGame.py -f 120 -i ml_play_template.py -i ml_play_template.py Maze_Car 2 MAZE 1 off`


![](https://i.imgur.com/ubPC8Fp.jpg)

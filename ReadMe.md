#include <Novice.h>
#include <stdlib.h>
#include <time.h>
#define _USE_MATH_DEFINES
#include <math.h>

const char kWindowTitle[] = "GC1D_06_サティッピッタヤユット＿サハチャート_タイトル";
const int kWindowWidth = 1280; //screen width
const int kWindowHeight = 720; //screen height

//enum 変数 プレイヤー向き方向
enum Direction {  //Direction 名をつける　use num to point direction
	Right,
	Left
};

// Windowsアプリでのエントリーポイント(main関数)
int WINAPI WinMain(HINSTANCE, HINSTANCE, LPSTR, int) {

	// ライブラリの初期化
	Novice::Initialize(kWindowTitle, kWindowWidth, kWindowHeight);

	///////////////////////////////////////////////////
	/////////////////   RANDOM TIME   /////////////////
	///////////////////////////////////////////////////

	unsigned int currentTime = (int)time(nullptr);
	srand(currentTime);

	///////////////////////////////////////////////////
	///////////////    BACKGROUND    //////////////////
	///////////////////////////////////////////////////

	int topBackground = 80;
	int downBackground = 640;

	///////////////////////////////////////////////////
	///////////////////   PLAYER   ////////////////////
	///////////////////////////////////////////////////

	float playerPosX = kWindowWidth / 2; //starting position
	float playerPosY = kWindowHeight / 2;//starting position
	float playerHeight = 100;
	float playerWidth = 100;
	float playerRadius = 65; //player hitbox
	int playerTexture = Novice::LoadTexture("./NoviceResources/white1x1.png");
	float playerMoveSpeed = 5;
	float playerMoveX = 0; //movement check
	float playerMoveY = 0;
	bool dash = 1; //dash function
	int dashCooldown = 0; //dash cooldown
	int playerDirection = Left; //player starting facing direction (ENUM

	//for drawQuad position (P)
	float qLeftTopX = -playerWidth / 2;   // O-
	float qLeftTopY = -playerHeight / 2;  // --

	float qRightTopX = playerWidth / 2;   // -O
	float qRightTopY = -playerHeight / 2; // --

	float qLeftDownX = -playerWidth / 2;  // --
	float qLeftDownY = playerHeight / 2;  // O-

	float qRightDownX = playerWidth / 2;  // --
	float qRightDownY = playerHeight / 2; // -O

	//rotate calculate for spinning effect
	float theta = 0; //rotate angle

	///////////////////////////////////////////////////
	/////////////////////   BEAN   ////////////////////
	///////////////  SPAWNING POSITION  ///////////////
	///////////////////////////////////////////////////

	int redBeanSmallPosX = kWindowWidth / 2;
	int redBeanSmallPosY = kWindowHeight / 3;
	float redBeanSmallRadius = 10;

	int redBeanMediumPosX = kWindowWidth / 2;
	int redBeanMediumPosY = kWindowHeight / 3;
	float redBeanMediumRadius = 20;

	int redBeanLargePosX = kWindowWidth / 2;
	int redBeanLargePosY = kWindowHeight / 3;
	float redBeanLargeRadius = 30;

	int specialBeanPosX = kWindowWidth / 2;
	int specialBeanPosY = kWindowHeight / 3;
	float specialBeanRadius = 10;

	///////////////////////////////////////////////////
	/////////////////////   BEAN   ////////////////////
	////////////////////  HIT BOX  ////////////////////
	///////////////////////////////////////////////////

	float smallBeanHitbox = 0;
	float mediumBeanHitbox = 0;
	float largeBeanHitbox = 0;
	float specialBeanHitbox = 0;

	///////////////////////////////////////////////////
	/////////////////   BEAN SPEED   //////////////////
	///////////////////////////////////////////////////

	int smallBeanSpeed = 3;
	int mediumBeanSpeed = 2;
	int largeBeanSpeed = 1;
	int specialBeanSpeed = 5;

	///////////////////////////////////////////////////
	/////////////   SPECIAL BEAN BONUS   //////////////
	///////////////////////////////////////////////////

	bool specialBonus = 0;
	int specialTime = 0;

	///////////////////////////////////////////////////
	////////////////   GAME FUNCTION   ////////////////
	///////////////////////////////////////////////////

	bool gameStart = 0;
	int frame = 0; //lot of use
	int hit = 0; //hit calculate
	int rotate = 0; //check rotate
	int time = 0;
	int timeAlive = 0;
	int score = 0;

	///////////////////////////////////////////////////

	// キー入力結果を受け取る箱
	char keys[256] = { 0 };
	char preKeys[256] = { 0 };

	// ウィンドウの×ボタンが押されるまでループ
	while (Novice::ProcessMessage() == 0) {
		// フレームの開始
		Novice::BeginFrame();

		// キー入力を受け取る
		memcpy(preKeys, keys, 256);
		Novice::GetHitKeyStateAll(keys);

		///
		/// ↓更新処理ここから
		///

		//reset//
		if (keys[DIK_SPACE] && !preKeys[DIK_SPACE] && gameStart == 0) {
			timeAlive = 0;
			score = 0;
			dashCooldown = 1;
		}

		///////////////////////////////////////////////////
		//////////////////  GAME START  ///////////////////
		///////////////////  TIME UP  /////////////////////
		//////////////////  FUNCTION  /////////////////////
		///////////////////////////////////////////////////

		if (keys[DIK_SPACE] && !preKeys[DIK_SPACE]) {
			gameStart = 1; // game start
		}
		if (gameStart == 1) { // time count down
			time = time - 1;
			if (time % 60 == 1) { //divided by 60 will count as 1
				timeAlive += 1;
			}
		}
		if (time == 0) { // when time 600 is 0
			gameStart = 0; //game stop
			time = 600; // add time
		}

		//////////////////////////   //////////////////////
		/////////////////////   STAGE   ///////////////////
		/////////////////////////  ////////////////////////

		if (playerPosX < 0 + playerWidth / 2) {
			playerPosX = 0 + playerWidth / 2;
		}
		if (playerPosY < topBackground + playerHeight / 2) {
			playerPosY = topBackground + playerHeight / 2;
		}
		if (playerPosX > kWindowWidth - playerWidth / 2) {
			playerPosX = kWindowWidth - playerWidth / 2;
		}
		if (playerPosY > kWindowHeight - playerHeight / 2) {
			playerPosY = kWindowHeight - playerHeight / 2;
		}

		///////////////////////////////////////////////////
		/////////////////////  PLAYER  ////////////////////
		///////////////////////////////////////////////////

		//player movement check
		playerMoveX = 0;
		playerMoveY = 0;

		//player control
		if (gameStart == 1) {
			if (keys[DIK_W]) {
				playerMoveY -= playerMoveSpeed;
			}
			if (keys[DIK_A]) {
				playerMoveX -= playerMoveSpeed;
				playerDirection = Left;
			}
			if (keys[DIK_S]) {
				playerMoveY += playerMoveSpeed;
			}
			if (keys[DIK_D]) {
				playerMoveX += playerMoveSpeed;
				playerDirection = Right;
			}
		}

		// 斜め移動 diagonal speed adjust
		if (playerMoveX != 0 && playerMoveY != 0) { //when AW WD DS SA is pressed
			playerMoveX /= sqrtf(2); //will use this speed instead
			playerMoveY /= sqrtf(2);
		}

		//斜め速度状態
		playerPosX += playerMoveX;
		playerPosY += playerMoveY;

		//for drawQuad new position (P') X = x*cos(theta) + y*sin(theta)
		//player use this for control    Y = y*cos(theta) - x*sin(theta)
		float qLeftTopRotatedY = qLeftTopY * cosf(theta) + qLeftTopX * sinf(theta) + playerPosY;
		float qLeftTopRotatedX = qLeftTopX * cosf(theta) - qLeftTopY * sinf(theta) + playerPosX;

		float qRightTopRotatedX = qRightTopX * cosf(theta) - qRightTopY * sinf(theta) + playerPosX;
		float qRightTopRotatedY = qRightTopY * cosf(theta) + qRightTopX * sinf(theta) + playerPosY;

		float qLeftDownRotatedY = qLeftDownY * cosf(theta) + qLeftDownX * sinf(theta) + playerPosY;
		float qLeftDownRotatedX = qLeftDownX * cosf(theta) - qLeftDownY * sinf(theta) + playerPosX;

		float qRightDownRotatedY = qRightDownY * cosf(theta) + qRightDownX * sinf(theta) + playerPosY;
		float qRightDownRotatedX = qRightDownX * cosf(theta) - qRightDownY * sinf(theta) + playerPosX;

		///////////////////////////////////////////////////
		/////////////////////   SPIN    ///////////////////
		///////////////////////////////////////////////////

		if (frame++ && gameStart == 1) { //if frame+
			rotate = 1; // it will set rotate to true
			if (frame % 1 == 0 && rotate == 1 && playerDirection == Left) {
				theta += 3; // rotate look faster
			}
			if (playerDirection == Right) {
				theta -= 3; // rotate look faster
			}
		}
		else rotate = 0; // if rotate is false

		///////////////////////////////////////////////////
		/////////////////////  DASH  //////////////////////
		///////////////////////////////////////////////////

		if (keys[DIK_SPACE] && !preKeys[DIK_SPACE]) {
			dash = 0; //make dash status from 1 to 0
		}
		if (dash == 0 && gameStart == 1) { //if dash status is 0
			dashCooldown = dashCooldown - 1; //dash time will countdown
		}
		if (dashCooldown == 0) { //if cooldown = 0 it will
			dashCooldown = 120; //add 120 to cooldown
			dash = 1; //make dash status form 0 to 1
		}
		if (dashCooldown >= 90 && dashCooldown < 120 && hit == 1 && gameStart == 1) { //make spin when hit
			if (frame % 1 == 0 && rotate == 1 && playerDirection == Left) {
				theta += 6;
			}
			if (playerDirection == Right) {
				theta -= 6;
			}
		}
		if (keys[DIK_W] && dashCooldown >= 90 &&
			dashCooldown < 120 && gameStart == 1) { //for 30 frame
			playerPosY -= playerMoveSpeed * 2; //go faster when dash 
		}
		if (keys[DIK_A] && dashCooldown >= 90 &&
			dashCooldown < 120 && gameStart == 1) {
			playerPosX -= playerMoveSpeed * 2;
		}
		if (keys[DIK_S] && dashCooldown >= 90 &&
			dashCooldown < 120 && gameStart == 1) {
			playerPosY += playerMoveSpeed * 2;
		}
		if (keys[DIK_D] && dashCooldown >= 90 &&
			dashCooldown < 120 && gameStart == 1) {
			playerPosX += playerMoveSpeed * 2;
		}

		///////////////////////////////////////////////////
		///////////////// Collision Detect ////////////////
		////////////////////  Install  ////////////////////
		///////////////////////////////////////////////////

		//length between bean and player
		smallBeanHitbox = sqrtf(powf(redBeanSmallPosX - playerPosX, 2) +
			powf(redBeanSmallPosY - playerPosY, 2));//small

		mediumBeanHitbox = sqrtf(powf(redBeanMediumPosX - playerPosX, 2) +
			powf(redBeanMediumPosY - playerPosY, 2));//medium

		largeBeanHitbox = sqrtf(powf(redBeanLargePosX - playerPosX, 2) +
			powf(redBeanLargePosY - playerPosY, 2));//large

		specialBeanHitbox = sqrtf(powf(specialBeanPosX - playerPosX, 2) +
			powf(specialBeanPosY - playerPosY, 2));//special

		///////////////////////////////////////////////////
		///////////////// Collision Detect ////////////////
		///////////////////////////////////////////////////

		//collision detection
		//small bean
		if (smallBeanHitbox <= playerRadius + redBeanSmallRadius) {
			hit = 1;
			time += 10; //time plus
			score += 50; //score plus
			redBeanSmallPosX = rand() % (kWindowWidth - 100); //respawn posX
			redBeanSmallPosY = rand() % (kWindowHeight - topBackground) //respawn posY
				+ topBackground;
		}
		//medium bean
		if (mediumBeanHitbox <= playerRadius + redBeanMediumRadius) {
			hit = 1;
			time += 30; //time plus
			score += 30; //score plus
			redBeanMediumPosX = rand() % (kWindowWidth - 100); //respawn posX
			redBeanMediumPosY = rand() % (kWindowHeight - topBackground) //respawn posY
				+ topBackground;
		}
		//large bean
		if (largeBeanHitbox <= playerRadius + redBeanLargeRadius) {
			hit = 1;
			time += 50; //time plus
			score += 10; //score plus
			redBeanLargePosX = rand() % (kWindowWidth - 100); //respawn posX
			redBeanLargePosY = rand() % (kWindowHeight - topBackground) //respawn posY
				+ topBackground;
		}
		//special bean
		if (specialBeanHitbox <= playerRadius + specialBeanRadius) {
			hit = 1;
			specialBeanPosX = rand() % (kWindowWidth - 100); //respawn posX
			specialBeanPosY = rand() % (kWindowHeight - topBackground) //respawn posY
				+ topBackground;
			specialBonus = 1; //got special bonus
			specialTime = 60; //in this time frame

			if (hit == 1) {
				dashCooldown = 120; //reset dash cooldown
				dash = 1;
			}
		}

		///////////////////////////////////////////////////
		//////////////////// BEAN MOVING //////////////////
		///////////////////////////////////////////////////

		//bean moving
		if (hit == 1 && gameStart == 1) {
			redBeanSmallPosX -= smallBeanSpeed;
			if (redBeanSmallPosX <= 0)
				redBeanSmallPosX = kWindowWidth;
		}
		//bean moving
		if (hit == 1 && gameStart == 1) {
			redBeanMediumPosX -= mediumBeanSpeed;
			if (redBeanMediumPosX <= 0)
				redBeanMediumPosX = kWindowWidth;
		}
		//bean moving
		if (hit == 1 && gameStart == 1) {
			redBeanLargePosX -= largeBeanSpeed;
			if (redBeanLargePosX <= 0)
				redBeanLargePosX = kWindowWidth;
		}
		//bean moving
		if (hit == 1 && gameStart == 1) {
			specialBeanPosX -= specialBeanSpeed;
			if (specialBeanPosX <= 0)
				specialBeanPosX = kWindowWidth;
		}

		///////////////////////////////////////////////////
		////////////// SPECIAL BONUS DETAIL ///////////////
		///////////////////////////////////////////////////

		if (specialBonus == 1) {
			specialTime = specialTime - 1;
			if (frame % 1 == 0 && rotate == 1 &&
				playerDirection == Left) {
				theta -= 8;
			}
			if (frame % 1 == 0 && rotate == 1 &&
				playerDirection == Right) {
				theta += 8;
			}
			if (specialTime == 0) {
				specialBonus = 0;
				specialTime += 60;
			}
			if (specialTime < 60 && specialTime > 0) {
				playerMoveSpeed = 8; //bonus speed
			}
		}
		else playerMoveSpeed = 5;

		///
		/// ↑更新処理ここまで
		///

		///
		/// ↓描画処理ここから
		///

		///////////////////////////////////////////////////
		///////////////    BACKGROUND    //////////////////
		///////////////////////////////////////////////////

		//background
		Novice::DrawBox(0, 80, kWindowWidth, downBackground,
			0.0f, BLACK, kFillModeSolid);

		///////////////////////////////////////////////////
		//////////////////   PLAYER   /////////////////////
		///////////////////////////////////////////////////

		//player vector
		Novice::DrawQuad((int)qLeftTopRotatedX, (int)qLeftTopRotatedY,
			(int)qRightTopRotatedX, (int)qRightTopRotatedY,
			(int)qLeftDownRotatedX, (int)qLeftDownRotatedY,
			(int)qRightDownRotatedX, (int)qRightDownRotatedY,
			(int)qLeftTopRotatedX, (int)qLeftTopRotatedY, (int)playerWidth, (int)playerHeight,
			playerTexture, WHITE);

		if (dash == 1) { //dash mark
			Novice::DrawEllipse((int)playerPosX, (int)playerPosY,
				(int)specialBeanRadius, (int)specialBeanRadius,
				0.0f, 0xda70d6, kFillModeSolid);
		}

		///////////////////////////////////////////////////
		///////////////////   BEAN   //////////////////////
		///////////////////////////////////////////////////

		//small bean
		Novice::DrawEllipse((int)redBeanSmallPosX, (int)redBeanSmallPosY,
			(int)redBeanSmallRadius, (int)redBeanSmallRadius,
			0.0f, RED, kFillModeSolid);

		//medium bean
		Novice::DrawEllipse((int)redBeanMediumPosX, (int)redBeanMediumPosY,
			(int)redBeanMediumRadius, (int)redBeanMediumRadius,
			0.0f, RED, kFillModeSolid);

		//special bean
		Novice::DrawEllipse((int)redBeanLargePosX, (int)redBeanLargePosY,
			(int)redBeanLargeRadius, (int)redBeanLargeRadius,
			0.0f, RED, kFillModeSolid);

		//large bean
		Novice::DrawEllipse((int)specialBeanPosX, (int)specialBeanPosY,
			(int)specialBeanRadius, (int)specialBeanRadius,
			0.0f, 0xda70d6, kFillModeSolid);

		///////////////////////////////////////////////////
		/////////////////////   TEXT   ////////////////////
		///////////////////////////////////////////////////

		//score
		Novice::ScreenPrintf(10, 10, "Score : %d",
			score);
		Novice::ScreenPrintf(10, 30,
			"Time Alive : %d", timeAlive);
		Novice::ScreenPrintf(10, 50, "Total Score : %d",
			score * timeAlive);

		//time//
		Novice::ScreenPrintf(kWindowWidth / 4, 10,
			"Press SPACE to start");
		Novice::ScreenPrintf(kWindowWidth / 4, 30,
			"Time : %d", time);

		//dash//
		Novice::ScreenPrintf(kWindowWidth / 2, 10,
			"Press SPACE to dash");
		Novice::ScreenPrintf(kWindowWidth / 2, 30,
			"Dash : %s", dash ? "READY" : "Not ready");
		Novice::ScreenPrintf(kWindowWidth / 2, 50,
			"Dash Cooldown : %d", dashCooldown);

		//message//
		Novice::ScreenPrintf(900, 10,
			"Try to stay alive as long as you can!");
		Novice::ScreenPrintf(900, 30,
			"The Bigger the Bean = Time plus");
		Novice::ScreenPrintf(900, 50,
			"The Smaller the Bean = Score plus");

		//time up//
		if (gameStart == 0) {
			if (playerPosY < 500) {
				Novice::ScreenPrintf((int)playerPosX - (int)playerWidth / 3,
					(int)playerPosY + (int)playerHeight,
					"TIME UP!");
				Novice::ScreenPrintf((int)playerPosX - (int)playerWidth,
					(int)playerPosY + (int)playerHeight + 20,
					"Press SPACE to restart");
			}
			else {
				Novice::ScreenPrintf((int)playerPosX - (int)playerWidth / 3,
					(int)playerPosY - (int)playerHeight,
					"TIME UP!");
				Novice::ScreenPrintf((int)playerPosX - (int)playerWidth,
					(int)playerPosY - (int)playerHeight + 20,
					"Press SPACE to restart");
			}
		}

		///
		/// ↑描画処理ここまで
		///

		// フレームの終了
		Novice::EndFrame();

		// ESCキーが押されたらループを抜ける
		if (preKeys[DIK_ESCAPE] == 0 && keys[DIK_ESCAPE] != 0) {
			break;
		}
	}

	// ライブラリの終了
	Novice::Finalize();
	return 0;
}

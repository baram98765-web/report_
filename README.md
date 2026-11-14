
## 공룡 달리기 게임

```
#define _CRT_SECURE_NO_WARNINGS
#include "raylib.h"
#include <stdlib.h>
#include <time.h>
#include <stdio.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 400

// 공룡 구조체
typedef struct {
    float x, y;
    float width, height;
    float velocityY;
    int isJumping;
    int isDuck;
} Dinosaur;

// 장애물 구조체
typedef struct {
    float x, y;
    float width, height;
    float speed;
    int active;
    int isBird;
} Obstacle;

// 공룡 그리기 함수
void DrawDinosaur(Dinosaur dino) {
    Color dinoColor;
    int x;
    int y;
    int frame;

    dinoColor = DARKGRAY;
    x = (int)dino.x;
    y = (int)dino.y;

    if (dino.isDuck) {
        DrawRectangle(x, y + 15, 50, 25, dinoColor);
        DrawRectangle(x + 35, y, 20, 20, dinoColor);
        DrawRectangle(x + 10, y + 40, 10, 10, dinoColor);
        DrawRectangle(x + 30, y + 40, 10, 10, dinoColor);
    }
    else {
        DrawRectangle(x + 10, y + 15, 25, 30, dinoColor);
        DrawRectangle(x + 20, y, 20, 20, dinoColor);
        DrawRectangle(x + 30, y + 5, 8, 8, WHITE);
        DrawRectangle(x + 5, y + 20, 10, 15, dinoColor);

        frame = (int)(GetTime() * 10);
        if (frame % 2 == 0) {
            DrawRectangle(x + 10, y + 45, 8, 15, dinoColor);
            DrawRectangle(x + 25, y + 45, 8, 10, dinoColor);
        }
        else {
            DrawRectangle(x + 10, y + 45, 8, 10, dinoColor);
            DrawRectangle(x + 25, y + 45, 8, 15, dinoColor);
        }
    }
}

// 장애물 그리기 함수
void DrawObstacle(Obstacle obs) {
    int ox;
    int oy;
    int frame;
    Vector2 p1, p2, p3, p4, p5, p6;

    ox = (int)obs.x;
    oy = (int)obs.y;

    if (obs.isBird) {
        DrawRectangle(ox, oy + 10, 30, 15, PURPLE);

        p1.x = (float)(ox + 5);  p1.y = (float)(oy + 10);
        p2.x = (float)(ox + 15); p2.y = (float)(oy - 5);
        p3.x = (float)(ox + 25); p3.y = (float)(oy + 10);
        p4.x = (float)(ox + 5);  p4.y = (float)(oy + 25);
        p5.x = (float)(ox + 15); p5.y = (float)(oy + 40);
        p6.x = (float)(ox + 25); p6.y = (float)(oy + 25);

        frame = (int)(GetTime() * 8);
        if (frame % 2 == 0) {
            DrawTriangle(p1, p2, p3, PURPLE);
        }
        else {
            DrawTriangle(p4, p5, p6, PURPLE);
        }
    }
    else {
        DrawRectangle(ox, oy, (int)obs.width, (int)obs.height, DARKGREEN);
        DrawRectangle(ox - 5, oy + 10, 6, 15, DARKGREEN);
        DrawRectangle(ox + (int)obs.width - 1, oy + 5, 6, 20, DARKGREEN);
    }
}

int main(void) {
    int state;
    Dinosaur dino;
    float groundY;
    float gravity;
    Obstacle obs;
    int score;
    float speedMultiplier;
    float dt;
    Rectangle dinoRect;
    Rectangle obsRect;
    char scoreText[32];

    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Raylib T-Rex Runner");
    SetTargetFPS(60);
    srand((unsigned int)time(NULL));

    state = 0;

    dino.x = 50.0f;
    dino.y = 275.0f;
    dino.width = 40.0f;
    dino.height = 55.0f;
    dino.velocityY = 0.0f;
    dino.isJumping = 0;
    dino.isDuck = 0;

    groundY = 275.0f;
    gravity = 1200.0f;

    obs.x = 800.0f;
    obs.y = 290.0f;
    obs.width = 25.0f;
    obs.height = 40.0f;
    obs.speed = 4.0f;
    obs.active = 0;
    obs.isBird = 0;

    score = 0;
    speedMultiplier = 1.0f;

    while (!WindowShouldClose()) {
        dt = GetFrameTime();

        if (state == 0) {
            if (IsKeyPressed(KEY_SPACE) || IsKeyPressed(KEY_UP)) {
                state = 1;
                score = 0;
                speedMultiplier = 1.0f;
                obs.active = 0;
            }
        }
        else if (state == 1) {
            score += (int)(dt * 10);
            speedMultiplier = 1.0f + ((float)score / 500.0f);

            if ((IsKeyPressed(KEY_SPACE) || IsKeyPressed(KEY_UP)) && !dino.isJumping) {
                dino.velocityY = -550.0f;
                dino.isJumping = 1;
                dino.isDuck = 0;
            }

            if (IsKeyDown(KEY_DOWN)) {
                dino.isDuck = 1;
                dino.height = 35.0f;
                dino.y = groundY + 20.0f;
            }
            else {
                if (dino.isDuck && !dino.isJumping) {
                    dino.isDuck = 0;
                    dino.height = 55.0f;
                    dino.y = groundY;
                }
            }

            if (dino.isJumping) {
                dino.y += dino.velocityY * dt;
                dino.velocityY += gravity * dt;

                if (dino.y >= groundY) {
                    dino.y = groundY;
                    dino.isJumping = 0;
                    dino.velocityY = 0.0f;
                    if (dino.isDuck) {
                        dino.height = 35.0f;
                        dino.y = groundY + 20.0f;
                    }
                }
            }

            if (!obs.active) {
                obs.active = 1;
                obs.x = (float)SCREEN_WIDTH + 50.0f;
                obs.isBird = ((rand() % 3 == 0) && (score > 200)) ? 1 : 0;
                if (obs.isBird) {
                    obs.y = groundY - 10.0f;
                    obs.width = 40.0f;
                    obs.height = 25.0f;
                }
                else {
                    obs.y = groundY + 15.0f;
                    obs.width = 25.0f;
                    obs.height = 40.0f;
                }
            }
            else {
                obs.x -= (350.0f * speedMultiplier) * dt;
                if (obs.x < -50.0f) {
                    obs.active = 0;
                }
            }

            dinoRect.x = dino.x + 5.0f;
            dinoRect.y = dino.y + 5.0f;
            dinoRect.width = dino.width - 10.0f;
            dinoRect.height = dino.height - 5.0f;

            obsRect.x = obs.x + 5.0f;
            obsRect.y = obs.y + 5.0f;
            obsRect.width = obs.width - 10.0f;
            obsRect.height = obs.height - 5.0f;

            if (CheckCollisionRecs(dinoRect, obsRect)) {
                state = 2;
            }
        }
        else if (state == 2) {
            if (IsKeyPressed(KEY_SPACE)) {
                state = 1;
                score = 0;
                dino.y = groundY;
                dino.isJumping = 0;
                dino.isDuck = 0;
                dino.height = 55.0f;
                obs.active = 0;
            }
        }

        BeginDrawing();
        ClearBackground(RAYWHITE);

        DrawLine(0, (int)(groundY + 55), SCREEN_WIDTH, (int)(groundY + 55), DARKGRAY);

        if (state != 0) {
            DrawDinosaur(dino);
            if (obs.active) {
                DrawObstacle(obs);
            }
        }

        if (state == 0) {
            DrawText("T-REX RUNNER (RAYLIB)", SCREEN_WIDTH / 2 - 180, SCREEN_HEIGHT / 2 - 40, 24, DARKGRAY);
            DrawText("Press [SPACE] or [UP] to Start", SCREEN_WIDTH / 2 - 140, SCREEN_HEIGHT / 2 + 10, 16, GRAY);
        }
        else if (state == 1) {
            sprintf(scoreText, "HI %05d", score);
            DrawText(scoreText, SCREEN_WIDTH - 150, 20, 20, DARKGRAY);
            DrawText("CONTROLS: [UP/SPACE] Jump  [DOWN] Duck", 20, 20, 14, GRAY);
        }
        else if (state == 2) {
            DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, Fade(BLACK, 0.4f));
            DrawText("G A M E   O V E R", SCREEN_WIDTH / 2 - 110, SCREEN_HEIGHT / 2 - 30, 28, RED);
            DrawText("Press [SPACE] to Restart", SCREEN_WIDTH / 2 - 100, SCREEN_HEIGHT / 2 + 15, 16, WHITE);
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

## 소감

```
외부 이미지 파일 없이 Raylib의 기본 도형 함수들만 활용하여 구글 공룡 게임의 핵심 메커니즘을 직접 구현해 보는 좋은 경험이었습니다.
C언어의 엄격한 문법과 구조체 등을 실제 게임 루프에 적용하면서 프로그래밍 구조에 대한 이해도를 크게 높일 수 있었고,
Visual Studio 환경에서 발생하는 컴파일 제약과 충돌 검사 로직을 직접 디버깅하며 완성도 있는 결과물을 만들어내어 큰 성취감을 느꼈습니다.
앞으로도 더 열심히 노력하겠습니다.
```

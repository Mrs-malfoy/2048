# 2048
my first project
#include <conio.h>
#include <windows.h>
#include <stdio.h>
#include <time.h>
//👇不知道在哪用到的
#include <io.h>
#include <direct.h>
#include <Shlobj.h>
#include <stdlib.h>

static char config_path[4096] = {0};//配置文件路径？
static void init_game();
static void loop_game();
static void release_game();
static int if_prepare_exit;//是否退出
static int if_game_over;//是否无路可走
static int if_need_add_num;
static int board[4][4];
static int score;
static int best;//???为什么要用static（静态全局变量不能被其他文件所用）
int main(int argc, char *argv[]) {
    init_game();//初始化游戏
    loop_game();//游戏循环
    release_game();//释放游戏
    return 0;
}
int read_keyboard() {
    return _getch();//<conio.h>
}
void clear_screen() {//!!!!!!!!!!重点理解，清屏！
    //👇坐标
    COORD pos = {0, 0};
    //👇设置光标位置
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
    CONSOLE_CURSOR_INFO info = {1, 0};
    //👇设置光标信息
    SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &info);
}
int get_null_count() {
    int n = 0;
    //to do
    for (int i = 0; i < 4; i++) {
        for (int j = 0; j < 4; j++) {
            if (board[i][j]) {
                n++;
            }
        }
    }
    return n;
}
void check_game_over() {
    //to do
    for (int i = 0; i < 4; i++) {
        for (int j = 0; j < 3; j++) {
            if (board[i][j] == board[i][j + 1] || board[j][i] == board[j + 1][i]) {
                if_game_over = 0;
                return ;
            }
        }
    }
    if_game_over = 1;
}
void refresh_show() {//刷新界面
    clear_screen();
    printf("\n\n\n\n");
    //to do 打印方格和数字
    printf("                  GAME: 2048     SCORE: %05d     BEST: %06d\n", score, best);
    printf("               --------------------------------------------------");
    //printf("\n\n                             ┌────┬────┬────┬────┐\n");
    printf("\n");
    for (int i = 0; i < 4; i++) {
        printf("                             |");
        for (int j = 0; j < 4; j++) {
            int value = board[i][j];
            if (value) {
                if (value < 10) {
                    printf("  %d |", value);
                } else if (value < 100) {
                    printf(" %d |", value);
                } else if (value < 1000) {
                    printf(" %d|", value);
                } else if (value < 10000) {
                    printf("%4d|", value);//?
                } else {
                    for (int k = 1; k < 20; k++) {
                        value >>= 1;
                        if (value == 1) {
                            printf("2^%02d|", k);
                            break;
                        }
                    }
                }
            } else printf("    |");
        }
        printf("\n");
        if (i < 3) {
        //    printf("\n                             ├────┼────┼────┼────┤\n");
        } else {
        //    printf("\n                             └────┴────┴────┴────┘\n");
        }
    }
    printf("\n");
    printf("               --------------------------------------------------\n");
    printf("                  [W]:UP [S]:DOWN [A]:LEFT [D]:RIGHT [Q]:EXIT");

    if (get_null_count() == 0) {//获取游戏面板上空位置的数量
        check_game_over();
        if (if_game_over) {
            printf("");
            CONSOLE_CURSOR_INFO info = {1, 1};
            SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &info);
        }
    }
    if (if_prepare_exit) {
        printf("");
        CONSOLE_CURSOR_INFO  info = {1, 1};
        SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &info);
    }
    fflush(0);//刷新输出缓冲区？
}
void add_rand_num() {//生成随机数
    srand((unsigned int) time(0));
    int n = rand() % get_null_count();
    //to do生成2或4
    for (int i = 0; i < 4; i++) {
        for (int j = 0; j < 4; j++) {
            if (!board[i][j] && !n) {
                if (rand() % 10) {
                    board[i][j] = 2;
                } else {
                    board[i][j] = 4;
                }
                return ;
            }
            n--;
        }
    }
}
void reset_game() {
    score = 0;
    if_need_add_num = 1;
    if_game_over = 0;
    if_prepare_exit = 0;

    int n = rand() % 16;
    //to do随机选择一个地方，赋值2
    for (int i = 0; i < 4; i++) {
        for (int j = 0; j < 4; j++) {
            if (!n) {
                board[i][j] = 2;
            }
            n--;
        }
    }
    add_rand_num();
    refresh_show();
}
void move_left() {
    //to do
    for (int i = 0; i < 4; i++) {
        int k = 0;
        for (int j = 1; j < 4; j++) {//非常之巧妙
            if (board[i][j]) {
                if (board[i][k] == board[i][j]) {
                    score += board[i][k++] *= 2;
                    board[i][j] = 0;
                    if_need_add_num = 1;
                } else if (board[i][k] == 0) {
                    board[i][k] = board[i][j];
                    board[i][j] = 0;
                    if_need_add_num = 1;
                } else {
                    board[i][++k] = board[i][j];
                    if (j != k) {
                        board[i][j] = 0;
                        if_need_add_num = 1;
                    }
                }
            }
        }
    }
}
void move_right() {
    for (int i = 0; i < 4; i++) {
        int k = 3;
        for (int j = 2; j >= 0; j--) {
            if (board[i][j]) {
                if (board[i][k] == board[i][j]) {
                    score += board[i][k--] *= 2;
                    board[i][j] = 0;
                    if_need_add_num = 1;
                } else if (board[i][k] == 0) {
                    board[i][k] = board[i][j];
                    board[i][j] = 0;
                    if_need_add_num = 1;
                } else {
                    board[i][--k] = board[i][j];
                    if (j != k) {
                        board[i][j] = 0;
                        if_need_add_num = 1;
                    }
                }
            }
        }
    }
}
void move_up() {
    for (int i = 0; i < 4; i++) {
        int k = 0;
        for (int j = 1; j < 4; j++) {//非常之巧妙
            if (board[j][i]) {
                if (board[k][i] == board[j][i]) {
                    score += board[k++][i] *= 2;
                    board[j][i] = 0;
                    if_need_add_num = 1;
                } else if (board[k][i] == 0) {
                    board[k][i] = board[j][i];
                    board[j][i] = 0;
                    if_need_add_num = 1;
                } else {
                    board[++k][i] = board[j][i];
                    if (j != k) {
                        board[j][i] = 0;
                        if_need_add_num = 1;
                    }
                }
            }
        }
    }
}
void move_down() {
    for (int i = 0; i < 4; i++) {
        int k = 3;
        for (int j = 2; j >= 0; j--) {
            if (board[j][i]) {
                if (board[k][i] == board[j][i]) {
                    score += board[k--][i] *= 2;
                    board[j][i] = 0;
                    if_need_add_num = 1;
                } else if (board[k][i] == 0) {
                    board[k][i] = board[j][i];
                    board[j][i] = 0;
                    if_need_add_num = 1;
                } else {
                    board[--k][i] = board[j][i];
                    if (j != k) {
                        board[j][i] = 0;
                        if_need_add_num = 1;
                    }
                }
            }
        }
    }
}
void loop_game() {
    while(1) {
        int cmd = read_keyboard();//接受标准输入流字符命令
        if (if_prepare_exit) {
            if (cmd == 'y') {
                clear_screen();
                return ;
            } else if (cmd == 'n') {
                //取消退出
                if_prepare_exit = 0;
                refresh_show();
                continue;
            } else {
                continue;
            }
        }

        if (if_game_over) {
            if (cmd == 'y') {
                reset_game();
                continue;
            } else if (cmd == 'n') {
                clear_screen();
                return ;
            } else {
                continue;
            }
        }

        if_need_add_num = 0;
        switch (cmd) {
            case 'a' : {move_left();break;}
            case 's' : {move_down();break;}
            case 'w' : {move_up();break;}
            case 'd' : {move_right();break;}
            case 'q' : {if_prepare_exit = 1;break;}
            default : continue;
        }
        //打破得分记录！！！！！！
        if (score > best) {
            best = score;
            //const char *config_path;
            FILE *fp = fopen(config_path, "w");
            if (fp) {
                fwrite(&best, sizeof(best), 1, fp);
                fclose(fp);
            }
        }
        if (if_need_add_num) {
            add_rand_num();//添加一个数字
            refresh_show();
        } else if (if_prepare_exit) {
            refresh_show();
        }
    }
}
void release_game(int signal) {
    system("cls");
    CONSOLE_CURSOR_INFO info = {1, 1};
    SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &info);
    exit(0);
}
void init_game() {
    system("cls");
    char m_lpszDefaultDir[MAX_PATH];
    char szDocument[MAX_PATH] = {0};
    memset(m_lpszDefaultDir, 0, _MAX_PATH);
    LPITEMIDLIST pidl = NULL;
    SHGetSpecialFolderLocation(NULL, CSIDL_LOCAL_APPDATA, &pidl);
    if (pidl && SHGetPathFromIDList(pidl, szDocument)) {
        GetShortPathName(szDocument, m_lpszDefaultDir, _MAX_PATH);
    }
    sprintf(config_path, "%s\\2048", m_lpszDefaultDir);
    if (_access(config_path, 0) == -1) {
        _mkdir(config_path);
    }
    sprintf(config_path, "%s\\2048\\2048.dat", m_lpszDefaultDir);


    /* 读取游戏最高分数 */
    FILE *fp = fopen(config_path, "r");
    if (fp) {
        fread(&best, sizeof(best), 1, fp);
        fclose(fp);
    } else {
        best = 0;
        fp = fopen(config_path, "w");
        if (fp) {
            fwrite(&best, sizeof(best), 1, fp);
            fclose(fp);
        }
    }

    reset_game();
    //没打完，太恶心了……
}
//现在，让我们来总结一下：

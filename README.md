
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define DICE_COUNT 5
#define CATEGORY_COUNT 12
#define PLAYER_COUNT 2

const char *category_names[CATEGORY_COUNT] = {
    "ONES", "TWOS", "THREES", "FOURS",
    "FIVES", "SIXES", "CHOICE",
    "FOUR OF A KIND", "FULL HOUSE",
    "SMALL STRAIGHT", "LARGE STRAIGHT", "YACHT"
};

// サイコロを振る
void roll_dice(int dice[], int keep[]) {
    for (int i = 0; i < DICE_COUNT; i++) {
        if (!keep[i]) {
            dice[i] = rand() % 6 + 1;
        }
    }
}

// サイコロをASCIIアートで表示
void show_dice(int dice[]) {
    const char *face[6][5] = {
        {"┌─────┐", "│     │", "│  ●  │",
         "│     │", "└─────┘"},
        {"┌─────┐", "│●    │", "│     │",
         "│    ●│", "└─────┘"},
        {"┌─────┐", "│●    │", "│  ●  │",
         "│    ●│", "└─────┘"},
        {"┌─────┐", "│●   ●│", "│     │",
         "│●   ●│", "└─────┘"},
        {"┌─────┐", "│●   ●│", "│  ●  │",
         "│●   ●│", "└─────┘"},
        {"┌─────┐", "│●   ●│", "│●   ●│",
         "│●   ●│", "└─────┘"}
    };

    printf("\n");
    for (int row = 0; row < 5; row++) {
        for (int i = 0; i < DICE_COUNT; i++) {
            printf("%s  ", face[dice[i] - 1][row]);
        }
        printf("\n");
    }

    printf("  1        2        3        4        5\n");
}

// 同じ目の個数を数える
int count_number(int dice[], int number) {
    int count = 0;

    for (int i = 0; i < DICE_COUNT; i++) {
        if (dice[i] == number) {
            count++;
        }
    }

    return count;
}

// 得点計算
int calculate_score(int dice[], int category) {
    int score = 0;

    // 1～6の目
    if (category >= 0 && category <= 5) {
        int number = category + 1;

        for (int i = 0; i < DICE_COUNT; i++) {
            if (dice[i] == number) {
                score += number;
            }
        }
    }

    // CHOICE：すべての目の合計
    else if (category == 6) {
        for (int i = 0; i < DICE_COUNT; i++) {
            score += dice[i];
        }
    }

    // FOUR OF A KIND：同じ目が4個以上
    else if (category == 7) {
        for (int n = 1; n <= 6; n++) {
            if (count_number(dice, n) >= 4) {
                for (int i = 0; i < DICE_COUNT; i++) {
                    score += dice[i];
                }
                break;
            }
        }
    }

    // FULL HOUSE：3個と2個の組み合わせ
    else if (category == 8) {
        int three = 0;
        int two = 0;

        for (int n = 1; n <= 6; n++) {
            int count = count_number(dice, n);

            if (count == 3) three = 1;
            if (count == 2) two = 1;
        }

        if (three && two) {
            score = 25;
        }
    }

    // SMALL STRAIGHT：連続する4種類
    else if (category == 9) {
        for (int start = 1; start <= 3; start++) {
            int found = 1;

            for (int n = start; n < start + 4; n++) {
                if (count_number(dice, n) == 0) {
                    found = 0;
                }
            }

            if (found) {
                score = 15;
                break;
            }
        }
    }

    // LARGE STRAIGHT：1～5 または 2～6
    else if (category == 10) {
        int has1 = count_number(dice, 1) == 1;
        int has2 = count_number(dice, 2) == 1;
        int has3 = count_number(dice, 3) == 1;
        int has4 = count_number(dice, 4) == 1;
        int has5 = count_number(dice, 5) == 1;
        int has6 = count_number(dice, 6) == 1;

        if ((has1 && has2 && has3 && has4 && has5) ||
            (has2 && has3 && has4 && has5 && has6)) {
            score = 30;
        }
    }

    // YACHT：5個すべて同じ目
    else if (category == 11) {
        if (count_number(dice, dice[0]) == 5) {
            score = 50;
        }
    }

    return score;
}

// 得点表
void show_score(int score[PLAYER_COUNT][CATEGORY_COUNT]) {
    printf("\n========== SCORE TABLE ==========\n");
    printf("%-20s %6s %6s\n", "CATEGORY", "P1", "P2");

    for (int i = 0; i < CATEGORY_COUNT; i++) {
        printf("%-20s", category_names[i]);

        for (int p = 0; p < PLAYER_COUNT; p++) {
            if (score[p][i] == -1) {
                printf("%6s", "--");
            } else {
                printf("%6d", score[p][i]);
            }
        }
        printf("\n");
    }

    printf("=================================\n");
}

// 整数を入力
int read_integer(void) {
    char line[100];
    char *end;
    long value;

    while (1) {
        if (fgets(line, sizeof(line), stdin) == NULL) {
            exit(0);
        }

        value = strtol(line, &end, 10);

        if (end != line) {
            while (*end == ' ' || *end == '\t' ||
                   *end == '\n' || *end == '\r') {
                end++;
            }

            if (*end == '\0') {
                return (int)value;
            }
        }

        printf("数字を入力してください：");
    }
}

// 残すサイコロを選択
void select_dice(int keep[]) {
    char line[100];
    char *p;
    char *end;
    int number;
    int valid = 1;

    while (1) {
        printf("残すサイコロの番号（例：1 3 5）\n");
        printf("全部振り直す場合は 0：");

        if (fgets(line, sizeof(line), stdin) == NULL) {
            exit(0);
        }

        for (int i = 0; i < DICE_COUNT; i++) {
            keep[i] = 0;
        }

        p = line;
        valid = 1;
        int entered = 0;

        while (*p != '\0') {
            while (*p == ' ' || *p == '\t' ||
                   *p == '\n' || *p == '\r') {
                p++;
            }

            if (*p == '\0') break;

            number = (int)strtol(p, &end, 10);

            if (end == p) {
                valid = 0;
                break;
            }

            if (number < 0 || number > DICE_COUNT) {
                valid = 0;
                break;
            }

            if (number == 0) {
                // 0だけを入力した場合は全部振り直す
                while (*end == ' ' || *end == '\t') end++;

                if (*end == '\n' || *end == '\r' ||
                    *end == '\0') {
                    entered = 1;
                    break;
                }

                valid = 0;
                break;
            }

            keep[number - 1] = 1;
            entered = 1;
            p = end;
        }

        if (valid && entered) {
            return;
        }

        printf("入力が正しくありません。もう一度入力してください。\n");
    }
}

// カテゴリ選択
int select_category(int score[], int dice[]) {
    int category;

    while (1) {
        printf("\nカテゴリを選択してください。\n");

        for (int i = 0; i < CATEGORY_COUNT; i++) {
            if (score[i] == -1) {
                printf("%2d : %-20s %2d点\n",
                       i + 1, category_names[i],
                       calculate_score(dice, i));
            }
        }

        printf("番号：");
        category = read_integer() - 1;

        if (category >= 0 && category < CATEGORY_COUNT &&
            score[category] == -1) {
            return category;
        }

        printf("使用済み、または無効なカテゴリです。\n");
    }
}

// 1人分のターン
void play_turn(int player, int turn,
               int score[PLAYER_COUNT][CATEGORY_COUNT]) {
    int dice[DICE_COUNT];
    int keep[DICE_COUNT] = {0};

    printf("\n=================================\n");
    printf(" 第%dターン / 12  PLAYER %d\n",
           turn, player + 1);
    printf("=================================\n");

    // 最初のロール
    roll_dice(dice, keep);

    for (int roll = 1; roll <= 3; roll++) {
        printf("\n%d回目のロール\n", roll);
        show_dice(dice);

        if (roll == 3) break;

        select_dice(keep);
        roll_dice(dice, keep);
    }

    int category = select_category(score[player], dice);
    score[player][category] = calculate_score(dice, category);

    printf("\nPLAYER %d：%s を選択\n",
           player + 1, category_names[category]);
    printf("獲得点数：%d点\n", score[player][category]);
}

// メイン処理
int main(void) {
    int score[PLAYER_COUNT][CATEGORY_COUNT];
    int total[PLAYER_COUNT] = {0, 0};

    srand((unsigned int)time(NULL));

    // -1は未使用カテゴリ
    for (int p = 0; p < PLAYER_COUNT; p++) {
        for (int i = 0; i < CATEGORY_COUNT; i++) {
            score[p][i] = -1;
        }
    }

    printf("=================================\n");
    printf("          YACHT GAME\n");
    printf("       2 PLAYER MATCH\n");
    printf("=================================\n");
    printf("各プレイヤー12ターンです。\n");
    printf("Enterキーを押して開始してください。");

    char line[100];
    fgets(line, sizeof(line), stdin);

    // 全12ターン、各ターンで2人がプレイ
    for (int turn = 1; turn <= CATEGORY_COUNT; turn++) {
        for (int p = 0; p < PLAYER_COUNT; p++) {
            play_turn(p, turn, score);
            show_score(score);

            if (p == 0) {
                printf("\n次はPLAYER 2です。\n");
                printf("Enterキーを押してください。");
                fgets(line, sizeof(line), stdin);
            }
        }
    }

    // 最終集計
    for (int p = 0; p < PLAYER_COUNT; p++) {
        for (int i = 0; i < CATEGORY_COUNT; i++) {
            total[p] += score[p][i];
        }
    }

    printf("\n=================================\n");
    printf("             GAME OVER\n");
    printf("=================================\n");
    printf("PLAYER 1：%d点\n", total[0]);
    printf("PLAYER 2：%d点\n", total[1]);

    if (total[0] > total[1]) {
        printf("PLAYER 1 WIN!\n");
    } else if (total[1] > total[0]) {
        printf("PLAYER 2 WIN!\n");
    } else {
        printf("DRAW!\n");
    }

    return 0;
}

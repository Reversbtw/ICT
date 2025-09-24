# ICT
DZ po ICT
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>
#include "ui.h" 

uiEntry *nameEntry;
uiCombobox *raceCombo;
uiMultilineEntry *outputBox;
uiCheckbox *hardModeCheck;
const char* races[] = {"Человек", "Эльф", "Гном", "Орк"};
const char* raceEmojis[] = {"🧑", "🧝", "🧙", "👹"};

int onWindowClosing(uiWindow *w, void *data) {
    uiQuit();
    return 1;
}

void onGenerateClicked(uiButton *b, void *data) {
    const char *name = uiEntryText(nameEntry);
    
    if (strlen(name) == 0) {
        uiMultilineEntrySetText(outputBox, "⚠️ Ошибка: введите имя персонажа!");
        return;
    }

    int raceIndex = uiComboboxSelected(raceCombo);
    int hardMode = uiCheckboxChecked(hardModeCheck);

    int str = rand() % 20 + 1;
    int dex = rand() % 20 + 1;
    int intel = rand() % 20 + 1;

    switch (raceIndex) {
        case 0: break;  
        case 1: dex += 3; str -= 2; break;  
        case 2: intel += 3; dex -= 2; break;  
        case 3: str += 3; intel -= 2; break;  
    }

    if (hardMode) {
        str += 5;
        dex += 5;
        intel += 5;
    }

    int isEasterEgg = (strcmp(name, "/god") == 0);
    if (isEasterEgg) {
        str = dex = intel = 25;  
    }

    char result[512];
    if (isEasterEgg) {
        snprintf(result, sizeof(result),
            "👑 Режим бога 👑\n\n"
            "Имя: %s (GOD MODE)\n"
            "Раса: %s %s\n"
            "Сила: %d ★\n"
            "Ловкость: %d ★\n"
            "Интеллект: %d ★\n\n"
            "✨ У вас есть все! ✨",
            name, races[raceIndex], raceEmojis[raceIndex], str, dex, intel);
    } else {
        snprintf(result, sizeof(result),
            "Имя: %s\n"
            "Раса: %s %s\n"
            "Сила: %d\n"
            "Ловкость: %d\n"
            "Интеллект: %d%s",
            name, races[raceIndex], raceEmojis[raceIndex], 
            str, dex, intel,
            hardMode ? "\n\nРежим сложности: ВКЛЮЧЕН" : "");
    }

    uiMultilineEntrySetText(outputBox, result);
}

int main() {
    uiInitOptions options = {0};
    const char *err = uiInit(&options);
    if (err != NULL) {
        fprintf(stderr, "Ошибка инициализации: %s\n", err);
        uiFreeInitError(err);
        return 1;
    }

    srand(time(NULL));  

    uiWindow *window = uiNewWindow("Генератор персонажа RPG", 400, 350, 0);
    uiWindowSetMargined(window, 1);
    uiWindowOnClosing(window, onWindowClosing, NULL);

    uiBox *vbox = uiNewVerticalBox();
    uiBoxSetPadded(vbox, 1);
    uiWindowSetChild(window, uiControl(vbox));

    uiBoxAppend(vbox, uiControl(uiNewLabel("Имя персонажа:")), 0);
    nameEntry = uiNewEntry();
    uiBoxAppend(vbox, uiControl(nameEntry), 0);

    uiBoxAppend(vbox, uiControl(uiNewLabel("Раса:")), 0);
    raceCombo = uiNewCombobox();
    for (int i = 0; i < 4; i++) {
        uiComboboxAppend(raceCombo, races[i]);
    }
    uiBoxAppend(vbox, uiControl(raceCombo), 0);

    hardModeCheck = uiNewCheckbox("Сложный режим (+5 ко всем характеристикам)");
    uiBoxAppend(vbox, uiControl(hardModeCheck), 0);

    uiButton *generateBtn = uiNewButton("Создать персонажа");
    uiButtonOnClicked(generateBtn, onGenerateClicked, NULL);
    uiBoxAppend(vbox, uiControl(generateBtn), 0);

    outputBox = uiNewMultilineEntry();
    uiMultilineEntrySetReadOnly(outputBox, 1);
    uiBoxAppend(vbox, uiControl(outputBox), 1);

    uiControlShow(uiControl(window));
    uiMain();
    uiUninit();

    return 0;
}

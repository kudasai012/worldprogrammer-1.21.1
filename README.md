

```markdown
# WorldProgrammer

## Мод для программирования внутри Minecraft

**Пишите настоящий Java код прямо в игре. Без IDE. Без перезапуска. Без сложных настроек.**

---

## 📋 Содержание

- [О моде](#-о-моде)
- [Технические характеристики](#-технические-характеристики)
- [Системные требования](#-системные-требования)
- [Установка](#-установка)
- [Быстрый старт](#-быстрый-старт)
- [Архитектура мода](#-архитектура-мода)
- [Структура скрипта](#-структура-скрипта)
- [Система событий](#-система-событий)
- [Работа с игроком](#-работа-с-игроком)
- [Работа с миром](#-работа-с-миром)
- [Работа с предметами](#-работа-с-предметами)
- [Работа с сущностями](#-работа-с-сущностями)
- [Встроенный API (ScriptAPI)](#-встроенный-api-scriptapi)
- [Система маппингов](#-система-маппингов)
- [Потоки и асинхронность](#-потоки-и-асинхронность)
- [DSL скрипты](#-dsl-скрипты)
- [Примеры](#-примеры)
- [Частые ошибки и решения](#-частые-ошибки-и-решения)
- [Для разработчиков](#-для-разработчиков)
- [Лицензия](#-лицензия)

---

## 🎮 О моде

WorldProgrammer — это модификация для Minecraft, которая превращает игру в среду разработки. Вы открываете встроенный редактор кода, пишете Java код, нажимаете кнопку — и ваш код начинает работать прямо в мире Minecraft. Не нужно устанавливать IntelliJ IDEA, не нужно разбираться в Gradle, не нужно перезапускать игру. Написали код — он работает.

Мод компилирует ваш Java код в реальном времени с помощью встроенного в JDK компилятора `javax.tools.JavaCompiler`. Скомпилированные классы загружаются через `URLClassLoader` и вызываются при наступлении игровых событий: сломали блок, нажали клавишу, прошёл игровой тик.

WorldProgrammer поддерживает два формата скриптов. Первый — полноценные `.java` файлы с настоящим Java кодом, доступом ко всем классам Minecraft и стандартной библиотеке Java. Второй — упрощённые DSL скрипты с командами вроде `give diamond 5` или `spawn zombie`, которые не требуют знания Java вообще.

Каждый игрок на сервере имеет собственную базу данных скриптов. Скрипты одного игрока не влияют на скрипты другого. При смене мира скрипты загружаются автоматически из папки сохранения этого мира.

---

## 🔧 Технические характеристики

### Ядро и платформа

WorldProgrammer работает на модлоадере **Fabric**. Это лёгкий и быстрый загрузчик модов для Minecraft, который использует систему миксинов для модификации игрового кода. Fabric был выбран потому что он обеспечивает быстрый запуск, минимальное потребление ресурсов и совместимость с большинством других модов.

Мод **не совместим** с Forge, NeoForge, Quilt или любыми другими загрузчиками модов. Только Fabric.

### Версии

| Компонент | Версия | Описание |
|---|---|---|
| **Minecraft** | `1.21.1` | Версия игры для которой создан мод |
| **Fabric Loader** | `0.19.3` или выше | Загрузчик модов Fabric |
| **Fabric API** | `0.116.12+1.21.1` | Библиотека API от Fabric |
| **Fabric Loom** | `1.16-SNAPSHOT` | Gradle плагин для сборки Fabric модов |
| **Java** | `21` | Минимальная версия Java |
| **Mod Version** | `1.0.0` | Текущая версия WorldProgrammer |

### Маппинги

Мод собран с использованием **Official Mojang Mappings** (`loom.officialMojangMappings()`). Это означает, что в исходном коде мода используются официальные имена классов и методов от Mojang — разработчиков Minecraft. Например, `ServerLevel` вместо `ServerWorld`, `ServerPlayer` вместо `ServerPlayerEntity`, `Component.literal()` вместо `Text.literal()`.

Однако в production-среде (когда мод установлен на обычный клиент Minecraft) все классы и методы Minecraft обфусцированы. Класс `ServerLevel` становится `class_3218`, метод `getX()` становится `method_23317()`. Для решения этой проблемы WorldProgrammer содержит собственный **препроцессор маппингов**, который автоматически заменяет читаемые имена на обфусцированные перед компиляцией скрипта.

### Библиотеки и зависимости

**Fabric API** — основная зависимость. Мод использует следующие модули Fabric API:

- `fabric-command-api-v2` — для регистрации команды `/worldprogrammer`
- `fabric-lifecycle-events-v1` — для событий запуска и остановки сервера
- `fabric-networking-api-v1` — для синхронизации данных между клиентом и сервером
- `fabric-events-interaction-v0` — для отслеживания действий игрока (разрушение блоков)
- `fabric-entity-events-v1` — для отслеживания смерти сущностей

**javax.tools.JavaCompiler** — встроенный в JDK компилятор Java. Используется для компиляции `.java` скриптов в runtime. Именно поэтому для работы мода требуется **JDK**, а не JRE. JRE не содержит компилятор и скрипты не будут работать.

**Gson** — библиотека Google для работы с JSON. Используется для сохранения и загрузки конфигурации, правил мира и баз данных скриптов игроков. Gson входит в состав Minecraft и не требует отдельной установки.

### Сборка проекта

Проект собирается с помощью **Gradle** и плагина **Fabric Loom**. Вот ключевые параметры из `build.gradle`:

```groovy
plugins {
    id 'net.fabricmc.fabric-loom-remap' version "${loom_version}"
}

dependencies {
    minecraft "com.mojang:minecraft:${project.minecraft_version}"
    mappings loom.officialMojangMappings()
    modImplementation "net.fabricmc:fabric-loader:${project.loader_version}"
    modImplementation "net.fabricmc.fabric-api:fabric-api:${project.fabric_api_version}"
}

tasks.withType(JavaCompile).configureEach {
    it.options.release = 21
}
```

Параметр `loom.officialMojangMappings()` указывает Fabric Loom использовать официальные маппинги Mojang. Это даёт доступ к читаемым именам классов и методов в исходном коде мода.

Параметр `it.options.release = 21` гарантирует что код компилируется под Java 21.

---

## 💻 Системные требования

### Обязательные требования

| Требование | Минимум | Рекомендуется |
|---|---|---|
| **Java** | JDK 21 | JDK 21 (Adoptium/Oracle) |
| **RAM** | 4 GB | 6-8 GB |
| **Minecraft** | 1.21.1 | 1.21.1 |
| **Fabric Loader** | 0.16.0+ | 0.19.3+ |
| **Fabric API** | любая для 1.21.1 | 0.116.12+1.21.1 |

### Почему JDK а не JRE?

JRE (Java Runtime Environment) содержит только среду выполнения Java. JDK (Java Development Kit) содержит дополнительно компилятор `javac` и библиотеку `javax.tools`. WorldProgrammer использует `javax.tools.JavaCompiler` для компиляции скриптов прямо во время игры. Если у вас установлен только JRE, скрипты не будут компилироваться и вы увидите ошибку:

```
§c§lCOMPILER ERROR:§r JRE detected instead of JDK. Cannot compile.
```

### Как проверить что установлен JDK

Откройте командную строку и введите:

```cmd
javac --version
```

Если появилась версия (например `javac 21.0.3`) — у вас JDK. Если команда не найдена — установите JDK.

### Где скачать JDK 21

- **Eclipse Adoptium** (рекомендуется): https://adoptium.net/
- **Oracle JDK**: https://www.oracle.com/java/technologies/downloads/

---

## 📥 Установка

### Шаг 1: Установите Fabric

1. Скачайте Fabric Installer с https://fabricmc.net/use/installer/
2. Запустите установщик
3. Выберите версию Minecraft `1.21.1`
4. Нажмите Install

### Шаг 2: Установите Fabric API

1. Скачайте Fabric API для 1.21.1 с https://modrinth.com/mod/fabric-api
2. Поместите `.jar` файл в папку `mods/`

### Шаг 3: Установите WorldProgrammer

1. Скачайте `worldprogrammer-1.0.0.jar`
2. Поместите файл в папку `mods/`

### Шаг 4: Убедитесь что используется JDK

В настройках лаунчера Minecraft убедитесь что путь к Java указывает на JDK 21, а не на JRE.

### Расположение папки mods

```
Windows:  %APPDATA%\.minecraft\mods\
macOS:    ~/Library/Application Support/minecraft/mods/
Linux:    ~/.minecraft/mods/
```

---

## 🚀 Быстрый старт

### Ваш первый скрипт за 2 минуты

1. Запустите Minecraft с установленным модом
2. Откройте WorldProgrammer (через команду `/worldprogrammer` или назначенную клавишу)
3. Нажмите **New Script**
4. Назовите файл `HelloWorld.java`
5. Вставьте этот код:

```java
package com.example.worldprogrammer.scripts;

import net.minecraft.server.level.ServerLevel;
import net.minecraft.server.level.ServerPlayer;
import net.minecraft.core.BlockPos;
import net.minecraft.network.chat.Component;

public class HelloWorld {

    public static void onBlockBreak(ServerLevel world, ServerPlayer player, BlockPos pos) {
        player.sendSystemMessage(Component.literal("§a§lПривет, мир!"));
        player.sendSystemMessage(Component.literal("§7Вы сломали блок на координатах:"));
        player.sendSystemMessage(Component.literal(
            "§e  X: " + (int)player.getX() +
            "  Y: " + (int)player.getY() +
            "  Z: " + (int)player.getZ()
        ));
    }
}
```

6. Нажмите **Enable**
7. Сломайте любой блок в игре — в чате появится приветствие!

### Добавляем клавишу

1. В WorldProgrammer найдите ваш скрипт
2. Нажмите кнопку **Keybind**
3. Нажмите клавишу которую хотите привязать (например `G`)
4. Добавьте в код метод `onKeyPress`:

```java
public static void onKeyPress(ServerLevel world, ServerPlayer player, String key) {
    player.sendSystemMessage(Component.literal("§bВы нажали клавишу " + key + "!"));
}
```

Теперь при нажатии `G` в чате появится сообщение.

---

## 🏗️ Архитектура мода

### Как устроен WorldProgrammer изнутри

WorldProgrammer состоит из нескольких ключевых компонентов. Понимание их работы поможет вам писать более эффективные скрипты и понимать почему что-то работает или не работает.

### ScriptEngine — Сердце мода

`ScriptEngine` — это класс который отвечает за всё: компиляцию скриптов, их выполнение, маппинг имён, кэширование. Когда вы включаете скрипт, ScriptEngine делает следующее:

1. **Препроцессинг** — исходный код проходит через метод `preProcessSourceCode()`. Этот метод автоматически добавляет `package` если его нет, добавляет `import static` для ScriptAPI, и самое важное — заменяет все читаемые имена классов и методов Minecraft на обфусцированные.

2. **Компиляция** — препроцессированный код записывается во временный файл и компилируется с помощью `javax.tools.JavaCompiler`. Компилятору передаётся classpath содержащий все JAR файлы Minecraft, Fabric и модов.

3. **Загрузка** — скомпилированный `.class` файл загружается через `URLClassLoader`.

4. **Кэширование** — класс сохраняется в кэш `compiledClassCache` чтобы не перекомпилировать скрипт при каждом событии.

5. **Вызов** — когда происходит игровое событие, ScriptEngine ищет соответствующий метод в скомпилированном классе и вызывает его через Reflection.

### ScriptAPI — Библиотека функций

`ScriptAPI` — это набор статических методов которые упрощают типичные задачи. Вместо того чтобы писать 10 строк кода для спавна моба через Registry, вы пишете одну строку: `spawn(world, "zombie", x, y, z)`.

ScriptAPI компилируется как часть мода с правильными маппингами, поэтому все его методы работают гарантированно. Когда вы вызываете `spawn()` в скрипте, вы на самом деле вызываете метод который уже скомпилирован с правильными обфусцированными именами.

Все методы ScriptAPI автоматически доступны в скриптах благодаря строке `import static com.example.worldprogrammer.engine.ScriptAPI.*` которую препроцессор добавляет автоматически.

### RuleManager — Управление правилами

`RuleManager` хранит настройки мира (правила) и базы данных скриптов игроков. Каждый игрок имеет уникальный идентификатор. В однопользовательском режиме используется идентификатор `host_player` чтобы скрипты не терялись при смене UUID в offline режиме.

### NetworkHandler — Сетевое взаимодействие

`NetworkHandler` синхронизирует скрипты и правила между сервером и клиентом. Когда игрок заходит на сервер, ему отправляется полный список его скриптов. Когда игрок редактирует скрипт в GUI, изменения отправляются на сервер.

### Mixins — Перехват событий

Мод использует Mixin систему Fabric для перехвата игровых событий. Mixins — это способ модификации существующих классов Minecraft без изменения их исходного кода. Вот какие Mixin'ы использует WorldProgrammer:

| Mixin | Что перехватывает |
|---|---|
| `PlayerMixin` | Тик игрока, получение урона |
| `BlockMixin` | Установка и разрушение блоков |
| `MobSpawnMixin` | Спавн мобов |
| `WeatherMixin` | Изменение погоды |
| `WorldGenMixin` | Генерация мира |
| `FallingBlockEntityMixin` | Падающие блоки (гравитация) |
| `FluidStateMixin` | Поведение жидкостей |
| `ExplosionMixin` | Взрывы |
| `ServerPlayNetworkHandlerMixin` | Сетевые пакеты, чат |

---

## 📝 Структура скрипта

### Полная анатомия скрипта

Каждый Java скрипт в WorldProgrammer — это обычный Java класс с определёнными правилами. Разберём каждый элемент подробно.

### Package (Пакет)

```java
package com.example.worldprogrammer.scripts;
```

Пакет определяет «адрес» класса в файловой системе. В WorldProgrammer все скрипты должны находиться в пакете `com.example.worldprogrammer.scripts` или его подпакетах. Препроцессор автоматически исправит пакет если вы напишете его неправильно или забудете.

Вы **можете** вообще не писать строку `package` — препроцессор добавит её автоматически.

### Import (Импорты)

```java
import net.minecraft.server.level.ServerLevel;
import net.minecraft.server.level.ServerPlayer;
import net.minecraft.core.BlockPos;
import net.minecraft.network.chat.Component;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.item.Items;
import net.minecraft.world.level.block.Blocks;
import net.minecraft.world.level.block.state.BlockState;
import net.minecraft.world.entity.Entity;
import net.minecraft.world.entity.EntityType;
import net.minecraft.world.phys.Vec3;
import net.minecraft.core.particles.ParticleTypes;
```

Импорты подключают классы Minecraft к вашему скрипту. Без импорта вы не сможете использовать класс — получите ошибку `cannot find symbol`.

Препроцессор автоматически добавляет `import static com.example.worldprogrammer.engine.ScriptAPI.*` — это даёт вам доступ ко всем функциям ScriptAPI без явного указания `ScriptAPI.` перед каждым вызовом.

Вы также можете импортировать стандартные классы Java:

```java
import java.util.HashMap;
import java.util.Map;
import java.util.List;
import java.util.ArrayList;
import java.util.UUID;
import java.util.Random;
```

### Таблица основных классов Minecraft

| Класс | Что представляет | Когда нужен |
|---|---|---|
| `ServerLevel` | Мир на сервере | Всегда (параметр `world`) |
| `ServerPlayer` | Игрок на сервере | Всегда (параметр `player`) |
| `BlockPos` | Координаты блока (x, y, z целые) | Работа с блоками |
| `Component` | Текст с форматированием | Сообщения в чат |
| `ItemStack` | Стак предметов | Выдача предметов |
| `Items` | Все предметы игры | Создание ItemStack |
| `Blocks` | Все блоки игры | Сравнение блоков |
| `BlockState` | Состояние блока | Проверка типа блока |
| `Entity` | Любая сущность | Работа с мобами |
| `EntityType` | Тип сущности | Спавн мобов |
| `Vec3` | Вектор (x, y, z дробные) | Направления, скорости |
| `ParticleTypes` | Все типы частиц | Визуальные эффекты |

### Class (Объявление класса)

```java
public class MyScript {
```

Имя класса **должно** совпадать с именем файла. Если файл называется `CoolScript.java`, то класс должен называться `CoolScript`. Нарушение этого правила приведёт к ошибке компиляции.

Класс должен быть `public`. Все методы-события должны быть `public static`.

### Переменные класса

Вы можете создавать статические переменные для хранения данных между вызовами:

```java
public class MyScript {

    // Эта переменная сохраняется между вызовами событий
    private static int counter = 0;

    // Map для хранения данных по игрокам
    private static final Map<Integer, Boolean> playerData = new HashMap<>();

    public static void onBlockBreak(ServerLevel world, ServerPlayer player, BlockPos pos) {
        counter++;
        message(player, "§eВсего сломано блоков: " + counter);
    }
}
```

Данные в статических переменных живут пока скрипт включён. Если вы отключите и включите скрипт, класс перекомпилируется и все данные сбросятся.

---

## 📡 Система событий

Система событий — основа WorldProgrammer. Каждое событие — это метод в вашем классе, который автоматически вызывается когда в игре происходит определённое действие. Вы не вызываете эти методы сами — их вызывает ScriptEngine.

Вы можете реализовать любое количество событий в одном скрипте. Не обязательно реализовывать все — только те которые вам нужны. Если метод не найден, ничего страшного не произойдёт.

### onBlockBreak

**Когда вызывается:** Игрок сломал (разрушил) блок в мире.

**Сигнатура:**

```java
public static void onBlockBreak(ServerLevel world, ServerPlayer player, BlockPos pos)
```

**Параметры:**

| Параметр | Тип | Описание |
|---|---|---|
| `world` | `ServerLevel` | Мир в котором сломан блок. Через этот объект вы можете устанавливать блоки, спавнить мобов, получать информацию о мире. |
| `player` | `ServerPlayer` | Игрок который сломал блок. Через него можно отправлять сообщения, давать предметы, телепортировать. |
| `pos` | `BlockPos` | Координаты сломанного блока. Содержит целочисленные x, y, z. |

**Когда НЕ вызывается:**
- Блок разрушен взрывом
- Блок разрушен механизмом (поршень)
- Блок разрушен водой/лавой
- Игрок в креативе мгновенно разрушает блок (зависит от настроек)

**Подробный пример:**

```java
public static void onBlockBreak(ServerLevel world, ServerPlayer player, BlockPos pos) {
    int blockX = pos.getX();
    int blockY = pos.getY();
    int blockZ = pos.getZ();

    player.sendSystemMessage(Component.literal(
        "§eБлок сломан на: " + blockX + ", " + blockY + ", " + blockZ
    ));

    // Заменить сломанный блок на алмазный
    setBlock(world, "diamond_block", blockX, blockY, blockZ);
}
```

### onKeyPress

**Когда вызывается:** Игрок нажал клавишу которая привязана к этому скрипту через WorldProgrammer GUI.

**Сигнатура:**

```java
public static void onKeyPress(ServerLevel world, ServerPlayer player, String key)
```

**Параметры:**

| Параметр | Тип | Описание |
|---|---|---|
| `world` | `ServerLevel` | Текущий мир |
| `player` | `ServerPlayer` | Игрок нажавший клавишу |
| `key` | `String` | Название нажатой клавиши (например `"G"`, `"H"`, `"KEY_1"`) |

**Особенности:**
- Событие обрабатывается на сервере, но клавиша перехватывается на клиенте
- Между нажатием и обработкой может быть задержка в 1-2 тика (из-за сетевого пакета)
- Можно привязать до 4 клавиш на один скрипт
- Клавиша работает когда открыт обычный игровой экран (не GUI, не чат, не инвентарь)

**Подробный пример:**

```java
private static int selectedWeapon = 0;
private static final String[] WEAPONS = {"Стрела", "Огненный шар", "Снежок", "Череп визера"};

public static void onKeyPress(ServerLevel world, ServerPlayer player, String key) {
    selectedWeapon = (selectedWeapon + 1) % WEAPONS.length;
    message(player, "§6Выбрано: §f" + WEAPONS[selectedWeapon]);
}

public static void onBlockBreak(ServerLevel world, ServerPlayer player, BlockPos pos) {
    switch (selectedWeapon) {
        case 0: shootArrow(player); break;
        case 1: shootFireball(player); break;
        case 2: shootSnowball(player); break;
        case 3: shootWitherSkull(player); break;
    }
}
```

### onPlayerTick

**Когда вызывается:** Каждый серверный тик для каждого игрока. Minecraft работает со скоростью 20 тиков в секунду, поэтому этот метод вызывается 20 раз в секунду.

**Сигнатура:**

```java
public static void onPlayerTick(ServerLevel world, ServerPlayer player)
```

**⚠️ КРИТИЧЕСКИ ВАЖНО:** Этот метод вызывается 20 раз в секунду. Если вы напишете тяжёлый код внутри — сервер начнёт лагать. Правила:

1. **Не** создавайте потоки внутри `onPlayerTick`
2. **Не** делайте операции с большим количеством блоков каждый тик
3. Используйте счётчики чтобы делать тяжёлые операции реже
4. **Не** отправляйте сообщения в чат каждый тик — чат заспамится

**Подробный пример:**

```java
import java.util.HashMap;
import java.util.Map;

public class TickExample {

    private static final Map<Integer, Integer> ticks = new HashMap<>();

    public static void onPlayerTick(ServerLevel world, ServerPlayer player) {
        int id = System.identityHashCode(player);

        int tick = ticks.containsKey(id) ? ticks.get(id) + 1 : 1;
        ticks.put(id, tick);

        // Каждую секунду (20 тиков)
        if (tick % 20 == 0) {
            float health = getHealth(player);
            if (health < 10.0F) {
                heal(player, 1.0F);
            }
        }

        // Каждые 5 секунд (100 тиков)
        if (tick % 100 == 0) {
            message(player, "§7Прошло " + (tick / 20) + " секунд");
        }
    }
}
```

### onPlayerDamage

**Когда вызывается:** Игрок получил урон от любого источника (моб, падение, голод, лава, утопление, другой игрок и т.д.).

**Сигнатура:**

```java
public static void onPlayerDamage(ServerLevel world, ServerPlayer player)
```

**Подробный пример:**

```java
public static void onPlayerDamage(ServerLevel world, ServerPlayer player) {
    float health = getHealth(player);
    float maxHealth = getMaxHealth(player);
    float percent = (health / maxHealth) * 100;

    if (percent < 25) {
        message(player, "§4§l⚠ КРИТИЧЕСКОЕ ЗДОРОВЬЕ: " + (int)percent + "% ⚠");
        teleport(player, 0, 100, 0);
    } else if (percent < 50) {
        message(player, "§c⚠ Здоровье: " + (int)percent + "%");
    }
}
```

### onMobSpawn

**Когда вызывается:** Моб (сущность) заспавнился в мире. Включает и естественный спавн и спавн через команды/спавнеры.

**Сигнатура:**

```java
public static void onMobSpawn(ServerLevel world, Entity entity)
```

**Обратите внимание:** в этом событии нет параметра `player`. Если вам нужно отправить сообщение конкретному игроку, используйте `world.getPlayers()` для получения списка игроков.

**Подробный пример:**

```java
public static void onMobSpawn(ServerLevel world, Entity entity) {
    String entityType = entity.getType().toString().toLowerCase();

    if (entityType.contains("creeper")) {
        entity.discard();
    }
}
```

### onChatMessage

**Когда вызывается:** Игрок отправил сообщение в чат.

**Сигнатура:**

```java
public static boolean onChatMessage(ServerPlayer player, String message)
```

**Возвращаемое значение:**
- `true` — отменить сообщение (оно не появится в чате)
- `false` — пропустить сообщение (оно появится как обычно)

**Подробный пример:**

```java
public static boolean onChatMessage(ServerPlayer player, String message) {
    if (message.startsWith("!")) {
        String command = message.substring(1).toLowerCase().trim();

        switch (command) {
            case "heal":
                heal(player, 20.0F);
                message(player, "§aВы вылечены!");
                break;
            case "fly":
                launch(player, 5.0);
                message(player, "§bВзлёт!");
                break;
            default:
                message(player, "§cНеизвестная команда: " + command);
                break;
        }

        return true; // Скрываем команду из чата
    }

    return false; // Обычные сообщения пропускаем
}
```

### Сводная таблица событий

| Событие | Сигнатура | Описание |
|---|---|---|
| `onBlockBreak` | `(ServerLevel, ServerPlayer, BlockPos)` | Игрок сломал блок |
| `onKeyPress` | `(ServerLevel, ServerPlayer, String)` | Нажата привязанная клавиша |
| `onPlayerTick` | `(ServerLevel, ServerPlayer)` | Каждый тик (20 раз/сек) |
| `onPlayerDamage` | `(ServerLevel, ServerPlayer)` | Игрок получил урон |
| `onMobSpawn` | `(ServerLevel, Entity)` | Моб заспавнился |
| `onChatMessage` | `(ServerPlayer, String) → boolean` | Сообщение в чате |

---

## 👤 Работа с игроком

### Сообщения

```java
// Через ScriptAPI (рекомендуется)
message(player, "§aПривет!");

// Напрямую через Minecraft API
player.sendSystemMessage(Component.literal("§aПривет!"));
```

**Таблица цветов:**

| Код | Цвет | Код | Цвет |
|---|---|---|---|
| `§0` | Чёрный | `§8` | Тёмно-серый |
| `§1` | Тёмно-синий | `§9` | Синий |
| `§2` | Тёмно-зелёный | `§a` | Зелёный |
| `§3` | Бирюзовый | `§b` | Голубой |
| `§4` | Тёмно-красный | `§c` | Красный |
| `§5` | Фиолетовый | `§d` | Розовый |
| `§6` | Золотой | `§e` | Жёлтый |
| `§7` | Серый | `§f` | Белый |

| Код | Стиль |
|---|---|
| `§l` | **Жирный** |
| `§o` | *Курсив* |
| `§n` | Подчёркнутый |
| `§m` | ~~Зачёркнутый~~ |
| `§k` | Мерцающий |
| `§r` | Сброс |

### Координаты и позиция

```java
// Текущая позиция (дробные числа — точная позиция)
double x = player.getX();
double y = player.getY();
double z = player.getZ();

// Позиция блока под ногами (целые числа)
BlockPos blockPos = player.blockPosition();

// Направление взгляда (нормализованный вектор)
Vec3 look = player.getLookAngle();

// Высота глаз
double eyeY = player.getEyeY();

// Углы поворота камеры (в градусах)
float yaw = player.getYRot();
float pitch = player.getXRot();
```

### Здоровье и характеристики

```java
float health = player.getHealth();        // 0.0 - 20.0
float maxHealth = player.getMaxHealth();   // обычно 20.0
player.heal(5.0F);                        // вылечить
int hunger = player.getFoodData().getFoodLevel(); // 0-20
int expLevel = player.experienceLevel;    // уровень опыта
UUID uuid = player.getUUID();             // уникальный ID
```

### Состояние игрока

```java
isSprinting(player)    // Бежит (Ctrl)?
isSneaking(player)     // Крадётся (Shift)?
isSwimming(player)     // Плывёт?
isFlying(player)       // Летит?
isOnGround(player)     // На земле?
isInWater(player)      // В воде?
isInLava(player)       // В лаве?
isBlocking(player)     // Блокирует щитом?
isUsingItem(player)    // Использует предмет (ПКМ)?
isDrawingBow(player)   // Натягивает лук?
isHoldingBow(player)   // В руке лук?
isHolding(player, "diamond_sword")  // В руке конкретный предмет?
```

### Движение

```java
launch(player, 1.5);               // Подбросить вверх
pushForward(player, 2.0);          // Толкнуть вперёд
setVelocity(player, 0, 1.0, 0);    // Установить скорость
addVelocity(player, 0.5, 0.3, 0);  // Добавить к скорости
double speed = getSpeed(player);     // Горизонтальная скорость
teleport(player, 100, 65, 200);     // Телепортация
```

---

## 🌍 Работа с миром

### Блоки

```java
// Установить блок
setBlock(world, "diamond_block", 100, 65, 200);
setBlock(world, "air", 100, 65, 200);  // удалить блок

// Получить блок
BlockState state = world.getBlockState(new BlockPos(100, 65, 200));

// Проверить тип
if (state.getBlock() == Blocks.DIAMOND_ORE) { ... }
if (state.isAir()) { ... }

// Навигация по позициям
BlockPos pos = new BlockPos(100, 65, 200);
BlockPos up = pos.above();
BlockPos down = pos.below();
BlockPos shifted = pos.offset(3, -1, 5);
```

### Взрывы

```java
explode(world, x, y, z, 4.0F);                    // обычный
createExplosion(world, x, y, z, 8.0F, true);       // с огнём
createExplosion(world, x, y, z, 8.0F, false);      // без огня
```

Справочник силы взрыва: `1.0` = слабый, `4.0` = ТНТ, `6.0` = крипер, `12.0` = кровать в Незере.

### Звуки

```java
playSound(world, "entity.experience_orb.pickup", x, y, z);
playSound(world, "entity.lightning_bolt.thunder", x, y, z);
playSound(world, "entity.generic.explode", x, y, z);
playSound(world, "entity.player.levelup", x, y, z);
playSound(world, "block.anvil.land", x, y, z);
playSound(world, "entity.ender_dragon.growl", x, y, z);
```

### Частицы

```java
// sendParticles(тип, x, y, z, количество, разбросX, Y, Z, скорость)
world.sendParticles(ParticleTypes.FLAME,
    player.getX(), player.getY() + 1, player.getZ(),
    20, 0.5, 0.5, 0.5, 0.02);
```

| Константа | Визуал |
|---|---|
| `ParticleTypes.FLAME` | Обычное пламя |
| `ParticleTypes.SOUL_FIRE_FLAME` | Голубое пламя |
| `ParticleTypes.SMOKE` | Дым |
| `ParticleTypes.LARGE_SMOKE` | Большой дым |
| `ParticleTypes.EXPLOSION` | Маленький взрыв |
| `ParticleTypes.EXPLOSION_EMITTER` | Большой взрыв |
| `ParticleTypes.FLASH` | Вспышка |
| `ParticleTypes.END_ROD` | Белые искры |
| `ParticleTypes.HEART` | Сердечки |
| `ParticleTypes.BUBBLE_POP` | Пузырьки |
| `ParticleTypes.ASH` | Пепел |
| `ParticleTypes.PORTAL` | Частицы портала |
| `ParticleTypes.ENCHANT` | Частицы зачарования |

---

## 🎒 Работа с предметами

### Выдача предметов

```java
// Через ScriptAPI
giveItem(player, "diamond", 64);
giveItem(player, "netherite_sword", 1);
giveItem(player, "elytra", 1);

// Через Minecraft API
player.getInventory().add(new ItemStack(Items.DIAMOND, 64));
```

### Специальные предметы

```java
giveBow(player, "§6§lСупер Лук");   // Лук с именем + 64 стрелы
giveAutoBow(player);                 // Автоматический лук
```

### Проверка предметов

```java
if (isHoldingBow(player)) { ... }
if (isHolding(player, "diamond_sword")) { ... }
```

---

## 🐄 Работа с сущностями

### Спавн мобов

```java
spawn(world, "zombie", x, y, z);
spawn(world, "skeleton", x, y, z);
spawn(world, "creeper", x, y, z);
spawn(world, "pig", x, y, z);
spawn(world, "iron_golem", x, y, z);
spawn(world, "ender_dragon", x, y, z);
spawn(world, "wither", x, y, z);
spawn(world, "warden", x, y, z);
```

### Снаряды

```java
shootArrow(player);                  // Стрела
shootFastArrow(player, 5.0F);       // Быстрая стрела
shootArrowSpread(player, 5, 10);    // 5 стрел веером
shootFireball(player);               // Огненный шар
shootSnowball(player);               // Снежок
shootWitherSkull(player);            // Череп визера
```

### Уничтожение мобов

```java
killAll(world, "zombie");
killAll(world, "creeper");
killAll(world, "phantom");
```

### Эффекты зелий

```java
// applyEffect(player, "id", тики, уровень)
// 20 тиков = 1 секунда, уровень: 0=I, 1=II, 2=III

applyEffect(player, "speed", 200, 1);            // Скорость II на 10 сек
applyEffect(player, "strength", 600, 2);          // Сила III на 30 сек
applyEffect(player, "regeneration", 100, 0);      // Регенерация I на 5 сек
applyEffect(player, "night_vision", 6000, 0);     // Ночное зрение на 5 мин
applyEffect(player, "invisibility", 600, 0);      // Невидимость на 30 сек
applyEffect(player, "fire_resistance", 6000, 0);  // Огнеупорность на 5 мин
```

---

## 📚 Встроенный API (ScriptAPI)

Все функции доступны напрямую без префикса `ScriptAPI.` благодаря автоматическому `import static`.

### Полный справочник

| Функция | Описание |
|---|---|
| **Сообщения** | |
| `message(player, "текст")` | Сообщение в чат |
| `sendMessage(player, "текст")` | Алиас для message |
| **Предметы** | |
| `giveItem(player, "id", кол)` | Выдать предмет |
| `giveBow(player, "имя")` | Выдать лук с именем |
| `giveAutoBow(player)` | Выдать автолук |
| **Мир** | |
| `setBlock(world, "id", x, y, z)` | Поставить блок |
| `spawn(world, "тип", x, y, z)` | Заспавнить моба |
| `explode(world, x, y, z, сила)` | Взрыв |
| `createExplosion(world, x, y, z, сила, огонь)` | Взрыв с настройками |
| `playSound(world, "id", x, y, z)` | Звук |
| `killAll(world, "тип")` | Убить всех мобов типа |
| **Здоровье** | |
| `heal(entity, кол)` | Вылечить |
| `damage(entity, кол)` | Нанести урон |
| `getHealth(player)` | Текущее здоровье |
| `getMaxHealth(player)` | Макс. здоровье |
| `getHunger(player)` | Голод |
| `getLevel(player)` | Уровень опыта |
| **Движение** | |
| `teleport(entity, x, y, z)` | Телепортация |
| `launch(player, сила)` | Подбросить вверх |
| `pushForward(player, сила)` | Толкнуть вперёд |
| `setVelocity(player, x, y, z)` | Установить скорость |
| `addVelocity(player, x, y, z)` | Добавить скорость |
| `getSpeed(player)` | Горизонтальная скорость |
| `getLookDirection(player)` | Направление взгляда |
| **Стрельба** | |
| `shootArrow(player)` | Стрела |
| `shootFastArrow(player, скорость)` | Быстрая стрела |
| `shootArrowSpread(player, кол, разброс)` | Веер стрел |
| `shootFireball(player)` | Огненный шар |
| `shootSnowball(player)` | Снежок |
| `shootWitherSkull(player)` | Череп визера |
| **Эффекты** | |
| `applyEffect(player, "id", тики, уровень)` | Применить эффект |
| **Проверки** | |
| `isSprinting(player)` | Бежит? |
| `isSneaking(player)` | Крадётся? |
| `isSwimming(player)` | Плывёт? |
| `isFlying(player)` | Летит? |
| `isOnGround(player)` | На земле? |
| `isInWater(player)` | В воде? |
| `isInLava(player)` | В лаве? |
| `isBlocking(player)` | Блокирует щитом? |
| `isUsingItem(player)` | Использует предмет? |
| `isDrawingBow(player)` | Натягивает лук? |
| `isDrawingNamedBow(player, "имя")` | Натягивает лук с именем? |
| `isHolding(player, "id")` | Держит предмет? |
| `isHoldingBow(player)` | Держит лук? |
| `isEating(player)` | Ест? |
| `isDrinking(player)` | Пьёт зелье? |
| `getDrawPower(player)` | Сила натяжения (0-1) |
| `getMainHand(player)` | Предмет в основной руке |
| `getOffHand(player)` | Предмет в левой руке |

---

## 🗺️ Система маппингов

### Что такое маппинги и зачем они нужны

Когда Mojang выпускает Minecraft, весь Java код проходит через процесс обфускации. Обфускация заменяет читаемые имена классов и методов на бессмысленные короткие имена. Класс `ServerLevel` становится `class_3218`, метод `getHealth()` становится `method_6032`.

WorldProgrammer собран с Official Mojang Mappings — в исходном коде мода используются читаемые имена. Но скрипты компилируются в runtime когда все классы Minecraft в памяти имеют обфусцированные имена.

Поэтому WorldProgrammer содержит систему автоматического маппинга которая работает в три этапа:

1. **Маппинг классов** — `ServerLevel` → `class_3218`
2. **Маппинг методов** — `getHealth()` → `method_6032()`
3. **Маппинг полей** — `Items.DIAMOND` → `Items.field_xxxxx`

Всё это происходит автоматически. Вы пишете обычный читаемый код, а препроцессор заменяет имена перед компиляцией.

---

## ⏱️ Потоки и асинхронность

### Правила работы с потоками

1. `thread.setDaemon(true)` — **ВСЕГДА** ставьте
2. `world.getServer().execute(() -> { ... })` — для изменений мира
3. `Thread.sleep(50)` — 50мс = 1 игровой тик
4. **НЕ** создавайте потоки в `onPlayerTick`

### Пример с потоком

```java
public static void onKeyPress(ServerLevel world, ServerPlayer player, String key) {
    Thread t = new Thread(() -> {
        try {
            for (int i = 5; i > 0; i--) {
                final int sec = i;
                world.getServer().execute(() ->
                    message(player, "§e" + sec + "...")
                );
                Thread.sleep(1000);
            }
            world.getServer().execute(() -> {
                message(player, "§a§lПОЕХАЛИ!");
                launch(player, 3.0);
            });
        } catch (InterruptedException ignored) {}
    });
    t.setDaemon(true);
    t.start();
}
```

---

## 📜 DSL скрипты

Помимо Java, WorldProgrammer поддерживает упрощённые DSL скрипты. Файлы **без** расширения `.java`.

### Формат

```
on <событие>
    <команда> <аргументы>
endon
```

### Команды

| Команда | Пример |
|---|---|
| `message "текст"` | `message "§aПривет!"` |
| `give <предмет> <кол>` | `give diamond 5` |
| `spawn <тип>` | `spawn zombie` |
| `setblock <тип>` | `setblock gold_block` |
| `heal <кол>` | `heal 10` |
| `damage <кол>` | `damage 5` |
| `teleport <x> <y> <z>` | `teleport 0 100 0` |
| `sound <id>` | `sound entity.player.levelup` |
| `explode [сила]` | `explode 4` |
| `projectile <тип>` | `projectile fireball` |
| `effect <id> <сек> [ур]` | `effect speed 10 1` |
| `killall <тип>` | `killall zombie` |
| `cancel` | `cancel` |
| `repeat <кол> <команда>` | `repeat 5 give diamond 1` |

### Пример

```
on block_break
    message "§eВы сломали блок!"
    give diamond 1
    effect speed 10 1
endon
```

---

## 💡 Примеры

### Лечебный круг

```java
package com.example.worldprogrammer.scripts;

import net.minecraft.server.level.ServerLevel;
import net.minecraft.server.level.ServerPlayer;
import net.minecraft.core.BlockPos;

import java.util.HashMap;
import java.util.Map;

public class HealCircle {

    private static final Map<Integer, int[]> circles = new HashMap<>();

    public static void onKeyPress(ServerLevel world, ServerPlayer player, String key) {
        int id = System.identityHashCode(player);

        if (circles.containsKey(id)) {
            int[] center = circles.get(id);
            clearCircle(world, center[0], center[1], center[2]);
            circles.remove(id);
            message(player, "§7Лечебный круг убран.");
            return;
        }

        int cx = (int) player.getX();
        int cy = (int) player.getY() - 1;
        int cz = (int) player.getZ();

        buildCircle(world, cx, cy, cz);
        circles.put(id, new int[]{cx, cy, cz});
        message(player, "§a✚ Лечебный круг создан!");
    }

    public static void onPlayerTick(ServerLevel world, ServerPlayer player) {
        int id = System.identityHashCode(player);
        if (!circles.containsKey(id)) return;

        int[] center = circles.get(id);
        double dx = player.getX() - center[0];
        double dz = player.getZ() - center[2];
        double dist = Math.sqrt(dx * dx + dz * dz);

        if (dist <= 5 && getHealth(player) < getMaxHealth(player)) {
            heal(player, 0.5F);
        }
    }

    private static void buildCircle(ServerLevel world, int cx, int cy, int cz) {
        int radius = 5;
        for (int x = -radius; x <= radius; x++) {
            for (int z = -radius; z <= radius; z++) {
                double dist = Math.sqrt(x * x + z * z);
                if (dist >= radius - 0.5 && dist <= radius + 0.5) {
                    setBlock(world, "sea_lantern", cx + x, cy, cz + z);
                }
            }
        }
    }

    private static void clearCircle(ServerLevel world, int cx, int cy, int cz) {
        int radius = 5;
        for (int x = -radius; x <= radius; x++) {
            for (int z = -radius; z <= radius; z++) {
                double dist = Math.sqrt(x * x + z * z);
                if (dist >= radius - 0.5 && dist <= radius + 0.5) {
                    setBlock(world, "grass_block", cx + x, cy, cz + z);
                }
            }
        }
    }
}
```

### Фрост Волкер

```java
package com.example.worldprogrammer.scripts;

import net.minecraft.server.level.ServerLevel;
import net.minecraft.server.level.ServerPlayer;
import net.minecraft.core.BlockPos;
import net.minecraft.world.level.block.Blocks;

import java.util.HashMap;
import java.util.Map;

public class FrostWalker {

    private static final Map<Integer, Boolean> enabled = new HashMap<>();

    public static void onKeyPress(ServerLevel world, ServerPlayer player, String key) {
        int id = System.identityHashCode(player);

        if (enabled.containsKey(id)) {
            enabled.remove(id);
            message(player, "§7❄ Фрост Волкер §cвыключен");
        } else {
            enabled.put(id, true);
            message(player, "§b❄ Фрост Волкер §aвключён!");
        }
    }

    public static void onPlayerTick(ServerLevel world, ServerPlayer player) {
        int id = System.identityHashCode(player);
        if (!enabled.containsKey(id)) return;

        int px = (int) player.getX();
        int py = (int) player.getY() - 1;
        int pz = (int) player.getZ();
        int radius = 3;

        for (int dx = -radius; dx <= radius; dx++) {
            for (int dz = -radius; dz <= radius; dz++) {
                if (Math.sqrt(dx * dx + dz * dz) > radius) continue;
                BlockPos pos = new BlockPos(px + dx, py, pz + dz);
                if (world.getBlockState(pos).getBlock() == Blocks.WATER) {
                    world.setBlockAndUpdate(pos, Blocks.ICE.defaultBlockState());
                }
            }
        }
    }
}
```

### Гравитационная пушка

```java
package com.example.worldprogrammer.scripts;

import net.minecraft.server.level.ServerLevel;
import net.minecraft.server.level.ServerPlayer;
import net.minecraft.world.entity.Entity;

public class GravityGun {

    public static void onKeyPress(ServerLevel world, ServerPlayer player, String key) {
        double px = player.getX();
        double py = player.getY();
        double pz = player.getZ();
        int count = 0;

        for (Entity e : world.getAllEntities()) {
            if (e == null || e == player) continue;

            double dx = e.getX() - px;
            double dy = e.getY() - py;
            double dz = e.getZ() - pz;
            double dist = Math.sqrt(dx * dx + dy * dy + dz * dz);

            if (dist > 15 || dist < 1) continue;

            double force = 1.5 / dist;
            e.setDeltaMovement(-dx * force, -dy * force + 0.5, -dz * force);
            e.hurtMarked = true;
            count++;
        }

        message(player, "§d⚡ Притянуто сущностей: §f" + count);
    }
}
```

---

## ❗ Частые ошибки и решения

### COMPILATION FAILED

**Причина:** Синтаксическая ошибка в Java коде.

**Решение:** Проверьте скобки, точки с запятой, импорты. Прочитайте номер строки в ошибке.

### cannot find symbol

**Причина:** Класс или метод не импортирован / не существует.

**Решение:** Добавьте нужный `import` вверху файла. Проверьте правильность написания.

### JRE detected instead of JDK

**Причина:** Установлена только среда выполнения Java, без компилятора.

**Решение:** Установите JDK 21 с https://adoptium.net/

### Runtime Error / Script crashed

**Причина:** Ошибка во время выполнения (null, деление на ноль и т.д.).

**Решение:** Добавьте проверки `if (player != null)` и `try-catch`.

### Сервер зависает

**Причина:** Тяжёлая операция в главном потоке.

**Решение:** Вынесите в отдельный поток с `Thread.sleep(50)` между пакетами.

### Скрипт не обновляется

**Причина:** Старая версия в кэше.

**Решение:** Отключите (Disable) и включите (Enable) скрипт заново.

---

## 🛠️ Для разработчиков

### Сборка из исходников

```bash
git clone <repository>
cd worldprogrammer
./gradlew build
```

### Запуск в dev среде

```bash
./gradlew runClient
./gradlew runServer
```

### Структура проекта

```
src/main/java/com/example/worldprogrammer/
├── WorldProgrammer.java            # Главный класс мода
├── client/
│   └── WorldProgrammerClient.java  # Клиентская часть
├── command/
│   └── WorldProgrammerCommand.java # Команда /worldprogrammer
├── config/
│   ├── WorldRulesConfig.java       # Конфигурация
│   └── CustomScript.java           # Класс скрипта
├── engine/
│   ├── ScriptEngine.java           # Компилятор и исполнитель
│   ├── ScriptAPI.java              # API для скриптов
│   └── ScriptContext.java          # Контекст выполнения
├── network/
│   └── NetworkHandler.java         # Сетевые пакеты
├── rules/
│   ├── RuleManager.java            # Менеджер правил
│   └── builtin/                    # Встроенные правила
└── mixin/                          # Mixin классы
```

---

## 📄 Лицензия

MIT License

```
Copyright (c) 2026 fdfdjfsk

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

*WorldProgrammer v1.0.0 — Программируйте Minecraft изнутри.*
```

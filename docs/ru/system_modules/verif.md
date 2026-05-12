# Verifier

Модуль для управления верификацией устройств Matrix через протокол SAS (Short Authentication String). Позволяет просматривать список устройств аккаунта и верифицировать их для работы в зашифрованных комнатах.

Только понимайте, что бот верифицирует устройство локально.

## Команды

### `.devices`

**Доступ:** OWNER

Отображает список всех устройств, привязанных к аккаунту Matrix, с их статусом верификации.

```
.devices
# → 🔍 | Fetching devices...
# → 📱 | Your Devices:
#
#    🖥 | My Phone
#    └ QAZWSX | ❌ Unverified
#
#    🖥 | Bot Instance
#    └ EDCRFV | 🤖 (This Bot)
#
#    Use .verif [device_id] to verify a specific device.
```

**Статусы:**
- 🤖 (This Bot) — текущее устройство, на котором запущен юзербот
- ✅ Verified — устройство верифицировано, бот делится с устройством ключами
- ❌ Unverified — устройство не верифицировано

### `.verif <device_id>`

**Доступ:** OWNER

Запускает SAS-верификацию указанного устройства. Нельзя верифицировать само устройство бота.

**Процесс верификации:**

1. **Проверка** — бот проверяет существование устройства и его текущий статус
2. **Инициация** — отправляет SAS-запрос на целевое устройство:
   ```
   🛡 | Verification initiated for: QAZWSX
   ⏳ Please accept the request on that device.
   ```
3. **Эмодзи** — после принятия запроса на целевом устройстве, бот отображает 7 эмодзи для сверки:
   ```
   🔐 | SAS Emoji for device QAZWSX:
   
   🐶 | 🐱 | 🐭 | 🐹 | 🐰 | 🦊 | 🐻
   
   ⏳ Verification in progress...
   ```
4. **Ожидание** — бот ожидает результат верификации (таймаут 120 секунд)
5. **Результат**:
   - `success` — `✅ | Device QAZWSX verified successfully!`
   - `cancelled` — `❌ | Verification cancelled for device QAZWSX.`
   - `timeout` — `❌ | Verification failed for device QAZWSX.`

```
.verif QAZWSX
# → 🔍 | Checking device QAZWSX...
# → 🛡 | Verification initiated for: QAZWSX
#    ⏳ Please accept the request on that device.
# → 🔐 | SAS Emoji for device QAZWSX:
#    🐶 | 🐱 | 🐭 | 🐹 | 🐰 | 🦊 | 🐻
#    ⏳ Verification in progress...
# → ✅ | Device QAZWSX verified successfully!
```

## Проверка статуса

Использует два уровня проверки:
1. `device.trust >= TrustState.VERIFIED` — прямая проверка статуса из crypto store
2. `store.is_key_signed_by(target, signer)` — проверка cross-signing (подпись устройства мастер-ключом)

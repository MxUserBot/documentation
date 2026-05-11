Module for managing Matrix device verification via the SAS (Short Authentication String) protocol. Allows viewing the list of account devices and verifying them for use in encrypted rooms.

Just understand that the bot verifies the device locally.

## Commands

### `.devices`

**Access:** OWNER

Displays a list of all devices linked to the Matrix account with their verification status.

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

**Statuses:**
- 🤖 (This Bot) — current device running the userbot
- ✅ Verified — device is verified, the bot shares keys with it
- ❌ Unverified — device is not verified

### `.verif <device_id>`

**Access:** OWNER

Starts SAS verification for the specified device. Cannot verify the bot's own device.

**Verification process:**

1. **Check** — bot checks device existence and current status
2. **Initiation** — sends a SAS request to the target device:
   ```
   🛡 | Verification initiated for: QAZWSX
   ⏳ Please accept the request on that device.
   ```
3. **Emoji** — after the request is accepted on the target device, the bot displays 7 emoji for comparison:
   ```
   🔐 | SAS Emoji for device QAZWSX:

   🐶 | 🐱 | 🐭 | 🐹 | 🐰 | 🦊 | 🐻

   ⏳ Verification in progress...
   ```
4. **Waiting** — bot waits for the verification result (120 second timeout)
5. **Result**:
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

## Status Check

Uses two levels of verification:
1. `device.trust >= TrustState.VERIFIED` — direct status check from the crypto store
2. `store.is_key_signed_by(target, signer)` — cross-signing check (device signed by master key)

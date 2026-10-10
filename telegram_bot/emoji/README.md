# NosiFit Telegram custom emoji

The custom emoji set the NosiFit bot will use in its messages. The style follows the website: the Lucide-style round 2 px stroke from `web/app/static/js/icons/`, the dark rounded tile and line weight of the brand mark (`web/app/static/img/brand/`), and accent colours from `web/app/static/css/core/variables.css`.

| key | colour token | fallback |
|---|---|---|
| calories | `--nf-orange` | 🔥 |
| protein | `--nf-red` | 🥩 |
| fat | `--nf-yellow-soft` | 🥑 |
| carbs | `--nf-violet` | 🌾 |
| fiber | `--nf-green` | 🥬 |
| sugar | `--nf-violet-soft` | 🍬 |
| meal | `--nf-text-main` | 🍽 |
| water | `--nf-accent-water` | 💧 |
| training | `--nf-accent-training` | 🏋 |
| recovery | `--nf-accent-recovery` | ❤ |
| sleep | `--nf-blue-soft` | 🌙 |

`icons.json` is the single source of truth for this mapping.

## Layout

```
src/*.svg          editable glyph sources (100×100 grid, currentColor, stroke 10)
icons.json         key → colour, fallback emoji, custom_emoji_id (null until registered)
build.py           renders PNGs and the preview, then validates them (stdlib + headless Chromium)
png/color/*.png    brand tile + accent glyph, 100×100 RGBA
png/mono/*.png     white glyph on transparent, 100×100 RGBA
preview.html/.png  every icon on light and dark chat backgrounds, at 64, 24 and 20 px
```

To rebuild after editing a source, run `python telegram_bot/emoji/build.py`. It fails if any output is not a 100×100 8-bit RGBA PNG with a transparent corner.

## Two packs, pick one

- **color** (`needs_repainting = false`): uses the brand tile and accent colours. Each icon sits on its own dark tile, so it reads the same in light and dark chats.
- **mono** (`needs_repainting = true`): Telegram repaints these to the current text colour, so they blend into the text of both themes.

## Telegram requirements (checked 2026-10-10)

- **Asset format:** a static custom emoji is a 100×100 PNG or WEBP (official Stickers docs). `InputSticker.format = "static"` takes .PNG or .WEBP (Bot API 10.3, as documented in aiogram 3.31).
- **Creating the pack:** use `createNewStickerSet` with `sticker_type="custom_emoji"`, or `/newemojipack` in @Stickers. Each sticker needs at least one standard emoji in `emoji_list`. Use the `fallback` column.
- **Sending:** in HTML parse mode (this bot's default) write `<tg-emoji emoji-id="ID">FALLBACK</tg-emoji>`. The inner text must be a valid standard emoji. Telegram shows it wherever custom emoji can't render, such as notifications and some forwards.
- **Eligibility:** the bot can send custom emoji if it bought an additional username on Fragment. It can also send them in messages sent directly to private, group and supergroup chats if the bot's owner has Telegram Premium. If neither applies, Telegram shows only the fallback emoji.

## Remaining steps (not done yet)

1. Register one pack from `png/color/` or `png/mono/`, using the fallback emoji from `icons.json`.
2. Read the real IDs with `getCustomEmojiStickers` or by forwarding an emoji to the bot, then fill `custom_emoji_id` in `icons.json`.
3. Confirm the bot is eligible: a Fragment username, or an owner with Premium.
4. Only then replace the emoji literals in `telegram_bot/handlers/*` with `<tg-emoji>` tags.

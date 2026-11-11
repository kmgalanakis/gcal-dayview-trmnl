# Google Calendar Day View for Terminus

This repo holds a [Terminus](https://github.com/usetrmnl/terminus) extension that shows today's Google Calendar on an e-ink screen.
It reads your calendar from its secret iCal address, so it needs no Google API key or OAuth app.
I built it for a jailbroken Kindle Paperwhite, but it works on any model Terminus can render for.

The two files use Terminus's own export format, so Terminus can import them directly.

| File | Contents |
| --- | --- |
| `configuration.yml` | The extension settings, its form fields, the default data and the two exchanges. |
| `template.html.liquid` | The Liquid template. |

## What the screen shows

The extension has three layouts. You pick one with the **Layout** field.

- **timeline** is the default. It draws hour slots like Google Calendar's day view. Overlapping events sit side by side in columns. A legend names each calendar, and each calendar gets its own fill style: solid, outline or striped.
- **focus** uses the same hour slots, with a different header and footer. The header shows the date and a big countdown, such as "in 27 min", followed by the next meeting's title and time. During a meeting it shows "now" and the time left. The footer names tomorrow's first event.
- **agenda** shows cards for the current and the next event, then a list of the rest of the day.

All three layouts show all-day events in a row at the top.

The template does the calendar work in the browser that renders the screen:

- It expands repeating events, including `RRULE`, `EXDATE`, `RDATE` and moved instances.
- It converts every time to the timezone you set.
- It hides events you declined.
- It can hide timed events marked **Free**.

## How it works

Terminus fetches the calendar through two exchanges.

1. The first exchange asks for `text/calendar`. Terminus parses the feed into events, and Liquid passes a slim copy of them to the page.
2. The second exchange asks for `text/plain` and gets the raw feed as text. Terminus' iCal parser drops attendee responses, so the raw text is the only place where `PARTSTAT=DECLINED` survives. The template sends only the event blocks that mention `DECLINED` to the page.

Both exchanges use every address in `ical_url`. Terminus makes one request per address, so 3 calendars give 6 sources.
The template takes your email from each iCal address. Declined events are hidden in every calendar, even calendars from different Google accounts.

A script at the bottom of the template works out which events fall on today and draws the layout.

## Set it up

1. In Google Calendar, open **Settings**, choose your calendar, and go to **Integrate calendar**. Copy the **Secret address in iCal format**.
   Treat this address like a password. Anyone who has it can read your calendar.
2. Zip the two files. The zip must hold them at its top level, not inside a folder:

   ```bash
   zip extension.zip configuration.yml template.html.liquid
   ```

3. In Terminus, open **Extensions**, click **Upload**, and attach `extension.zip`. Terminus creates the extension with its fields, data and both exchanges.
4. Open the new extension. Under **Data**, replace the `ical_url` placeholder with your secret address.
   For several calendars, put up to 3 addresses in `ical_url`, separated by spaces.
5. Under **Build Matrix**, choose **Models** and select your device's model. The export format does not include this setting.
6. Save the extension, then click **Build**. Terminus renders a screen for each selected model.
7. Open your device's playlist and add the new screen.

## Settings

| Field | Default | Meaning |
| --- | --- | --- |
| `ical_url` | none | Your secret iCal address. Up to 3, separated by spaces. |
| `timezone` | `Europe/Athens` | IANA timezone name. |
| `clock` | `24h` | `24h` or `12h`. |
| `locale` | `en-GB` | Language for weekday and month names, such as `el-GR`. |
| `max_events` | `7` | Maximum number of events in the agenda list. |
| `hide_free` | `no` | Set to `yes` to hide timed events marked Free. |
| `title` | `Google Calendar` | Text in the title bar. |
| `layout` | `timeline` | `timeline`, `focus` or `agenda`. |
| `day_start` | `9` | First hour of the timeline. An earlier event stretches the grid. |
| `day_end` | `18` | Last hour of the timeline. A later event stretches the grid. |
| `calendar_names` | `Work, Personal` | One name for each iCal address, in the same order. Shown in the legend. |
| `calendar_styles` | `solid, outline, striped` | One style for each iCal address. |
| `my_email` | empty | Your emails, separated by spaces, used to find events you declined. When empty, the template takes one from each iCal address. Set it when a calendar's iCal address holds a calendar ID instead of your email. |

## Screen size

The layout fits the model "Amazon Kindle PW 7th Gen (Custom)": 1072×1448 pixels with a scale factor of 1.6.
Terminus divides the size by the scale factor, so the template has about 670×905 CSS pixels to fill.
For another model, compute the same CSS size and adjust the font sizes in the `<style>` block if needed.

## Keep the copies in sync

Terminus stores the live copy of the extension.
When you change the extension in Terminus, click its **Download** link on the **Extensions** page and unzip the export over these two files.
The export holds your real iCal addresses under `data`. Replace them with the placeholder before you commit, and never commit a real iCal address.

## License

MIT. See [LICENSE](LICENSE).

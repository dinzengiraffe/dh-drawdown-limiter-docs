# DH Drawdown Limiter — Parameter Reference (MQL5 Market version)

> **Note!** This reference applies to the MQL5 Market version. It contains three of the five reaction models (the two models that disable algo trading are reserved for the direct version) and no "LICENSE" input group.

## Loss Limit

| Variable | Default Value | Description |
|---|---|---|
| Behavior of this trigger | Off | Behavior of this trigger: off, warning only, or reaction (see values below). |
| Loss calculation basis | Percent of account balance | Calculation basis of the loss limit: percent of the account balance or a fixed money amount. |
| Reaction level or Set: Warn only (10 = 10% or 10 USD/EUR) | 10.0 | The threshold at which the reaction is triggered (with "Warn only": the warning threshold). |

- **Off** — This trigger is fully disabled. The configured threshold applies **nowhere anymore**.
  The scale does not disappear; it measures neutrally against the whole account (0 to 100%), and the tile heading carries `(OFF)`.
- **Warn only** — Only a warning is reported, and an all-clear once the situation has normalized.
- **React + early warning / all-clear** — The same as Warn only, with an additional red marker on the scale. When the marker is reached, positions and pending orders are closed (the magic-number and symbol filter is respected).
- **React only** — Positions and pending orders are closed (the magic-number and symbol filter is respected), but **without** a warning/all-clear message. The two markers for warning and all-clear also disappear from the scale.

Together, the four values cover every combination of "reports" and "acts":

| | reports warning/all-clear | acts |
|---|---|---|
| Off | no | no |
| Warn only | yes | no |
| React + early warning | yes | yes |
| React only | no | yes |

### Changing thresholds directly in the dashboard

- **Control row:** With the lock open, a control row appears below each pair of scales, divided horizontally into six areas (`◄` value `►`). Each area sits below the marker it belongs to. On the loss side of the equity scale the reaction therefore sits on the left, everywhere else on the right.
- **Input:** The value field is an input field: click into it, enter a number, press Enter. A value outside the allowed range is set to the nearest valid one (see the limits below), and the correction is noted in the journal. Letters are rejected, and the field jumps back to the previous value.
- **Fixed limits:** loss limit 0.1–100% (in money mode, 1 up to the account balance). Margin thresholds at least the broker's stop-out value and at most 100,000%. Below the broker's stop-out, the broker acts first, so a margin threshold of your own would never come into play there. The equity levels are bounded on both sides: the **reaction line** must keep one grid step away from the calculation basis (right on the basis itself there would be no span left to calculate from), and no level may sit lower than 0.1% of the calculation basis.
- **Apply and Discard:** Several thresholds can be open at once. Each is applied or discarded **individually**. Shortly after entry, the threshold's two arrows become the Apply and Discard buttons.
- **Closing the lock discards anything unconfirmed** and hides the control rows. **Collapsing the dashboard** closes the lock along with it and has the same effect.

## Equity Level

| Variable | Default Value | Description |
|---|---|---|
| Calculation basis: 0 = Current account balance | 0.0 | Calculation basis for both equity levels: 0 = the current account balance (the levels then move along with it), or a fixed value. |
| Unit of both levels: % or money distance | Percent | Unit for **both** levels: the distance from the calculation basis, in percent or in account currency. |
| Behavior lower to the Level calculation basis | Off | Behavior of the lower level. |
| Reaction level or Set: Warn only (distance BELOW the basis, % or money) | 10 | Distance **below** the calculation basis at which the reaction is triggered. 0 = unused. |
| Behavior upper the Level calculation basis | Off | Behavior of the upper level. |
| Reaction level or Set: Warn only (distance ABOVE the basis, % or money) | 10 | Distance **above** the calculation basis at which the reaction is triggered. 0 = unused. |

If both sides are "Off", the equity scale disappears from the dashboard.

### Equity scale, two directions

The equity scale is displayed outward from the center (the calculation basis) in different colors.
To the left of the center lies the loss area, with a color gradient toward red.
To the right of the center lies the profit area without risk, with a color gradient toward blue.

| | lower level (loss) | upper level (profit) |
| --- | --- | --- |
| Bar | green → white → yellow → orange → red | green → white → light blue → blue |
| Warning | orange | cyan blue |
| Reaction line | red, turns white once reached | deep blue, turns white once reached |
| All-clear | green | green |

### Percent or money amount

Both inputs are the **distance from the calculation basis**. The unit only determines how that distance is measured.

In **percent mode** (default), "10" for the lower level means 10% below it. With a calculation basis of 10,000, that is a level of 9,000. In **money mode**, "1,000" for the lower level means 1,000 below the basis, so again 9,000, and for the upper level 1,000 above it, so 11,000.

If the calculation basis is set to 0, it is the account balance, and the levels move along with it. That holds for both units: the percentage or the money distance stays, the resulting level shifts.

**For a fixed mark**, set a fixed calculation basis: a basis of 10,000 with a distance of 1,000 gives a level of 9,000 that stays there, whatever the account does. Keep in mind that the calculation basis is also the center of the dashboard's equity scale. That center then stays put as well.

A distance that reaches or exceeds the calculation basis is capped at 99.9%, so the level never ends up at zero or on the wrong side. In money mode, the Drawdown Limiter prints the two resulting levels to the journal at startup.

## Margin Protection

| Variable | Default Value | Description |
|---|---|---|
| Behavior of this trigger | Off | Behavior of this trigger. Runs independently of the symbol/magic filter. |
| Margin below this Level / Warn only: warning level | 200.0 | The margin level in percent whose **undercutting** triggers the reaction. |

**Margin protection has no filter.** The symbol and magic-number filter has no effect on it, and that is intentional: the margin level is a figure for the **whole account**. A position hidden by the filter still ties up margin and still brings the account closer to a stop-out.

**Special case: broker stop-out.** If the limiter's own margin protection does not react (behavior "Off" or "Warn only", in both cases the Drawdown Limiter itself takes no action) and the broker's stop-out value is known (the broker only reports it in percent mode), the tile and scale instead show this broker value, explicitly labeled "Broker Stop-Out". That is where all positions get closed by the broker itself, regardless of the Drawdown Limiter. With behavior "React", both instead show your own configured threshold.

## Filter (empty = include everything)

| Variable | Default Value | Description |
|---|---|---|
| Only these magic numbers, e.g. 12345,67890 (0 = manual) | *(empty)* | Only consider these magic numbers when monitoring. `0` also includes manually opened positions. |
| Only these symbols, e.g. EURUSD,XAUUSD.raw | *(empty)* | Only consider these symbols when monitoring. |
| Automatic pause (broker rollover) | true | Pause monitoring within the rollover window. |
| Window start: hour (server time, 0-23) | 23 | Start hour of the rollover window, server time. |
| Window start: minute (0-59) | 55 | Start minute of the rollover window. |
| Window end: hour (server time, 0-23) | 0 | End hour of the rollover window, server time. |
| Window end: minute (0-59) | 10 | End minute of the rollover window. |

If both magic numbers and symbols are entered, both criteria must match. The filter applies uniformly to the loss calculation, **equity**, closing, and the position table in the dashboard.

Specifically for equity: with a filter set, the EQUITY tile no longer shows `ACCOUNT_EQUITY`, but the account balance plus the floating result of only the filtered positions, and the **equity levels trigger on this value**. When using the filter together with equity levels, the thresholds should be reviewed accordingly. Account balance and margin level remain account-wide.

**Rollover window.** Around the daily rollover, spreads can briefly become extremely wide and distort equity without any actual market risk. In this daily recurring time window (server time, around midnight, 23:55 to 00:10), the Drawdown Limiter pauses automatically, as with the pause button, including the dashboard display and message, just set and released automatically. Pause means throughout: no reaction, no push and no popup (warning and all-clear stay off as well). The journal, however, still logs everything.

**A manually set pause remains in effect beyond the end of the window.** If you additionally paused it yourself, monitoring does **not** resume at 00:10. The Drawdown Limiter only lifts its own automatic pause, never yours. You switch the pause button in the dashboard off yourself.

## Reaction

| Variable | Default Value | Description |
|---|---|---|
| What happens after triggering | Close positions | What should happen after triggering (see values below). |
| What gets closed | Positions and pending orders | Scope of the intervention: **Positions and pending orders**, **Open positions only**, or **Pending orders only**. Applies to the reaction itself and to every reaction round while locked. |
| Lock: wait time after each close (sec.) | 60 | Only in the **locked state**: wait time between two reaction rounds. Without this wait time, the Drawdown Limiter would keep closing against an Expert Advisor that reopens immediately, and every round costs spread. Consequence of the wait time: a newly opened position stays open for up to this length of time. `0` = every round immediately. |

**What all three modes have in common:** on triggering, whatever is specified under "What gets closed" gets closed. The modes differ only in **what happens afterward**:

| Value | afterward | re-triggering (reaction level) |
| --- | --- | --- |
| Close positions whenever the limit is reached | back to normal monitoring until a trigger level is reached again | every time again |
| **Close positions** (default) | **lock**: every new position and pending order is closed on each reaction round | only after unlocking in the dashboard |
| Close positions and shut down the terminal | every new position and pending order is closed, and the terminal is shut down | — |

**The lock** (mode "Close positions") is the difference between closing positions once and keeping everything closed: every position and every pending order that appears afterward is closed again at the pace of the wait time. The lock persists across chart changes and terminal restarts and only ends when it is unlocked in the dashboard.

## Execution

| Variable | Default Value | Description |
|---|---|---|
| Magic number of the limiter | 970000 | Magic number of the orders sent by the Drawdown Limiter itself. For traceability in the journal only. |
| Retries per ticket | 3 | Retry attempts per ticket if closing or deleting fails. |
| Pause between retries (ms) | 300 | Wait time between two retry attempts. |
| Slippage: max. price deviation when closing (points) | 20 | Maximum allowed price deviation when closing. |

## Connection

| Variable | Default Value | Description |
|---|---|---|
| Monitor connection | true | Monitor the connection to the broker's server. |
| Only report once the connection has been down for X minutes | 2 | Minimum duration of a disruption before "connection disrupted" is reported. |
| All-clear only after X minutes without disruption | 1 | Minimum duration of a stable connection before the all-clear is reported. |

## Messages: Basic Settings

**What gets reported is decided by the following two groups, the same for both channels.** The journal is outside this rule: it is the complete record, cannot be turned off, and always logs the account values as well.

For push notifications, the terminal's own requirements additionally apply: a stored MetaQuotes ID and notifications switched on.

| Variable | Default Value | Description |
|---|---|---|
| Send push to the MT5 app | true | Send push notifications to the MT5 app. |
| Show a popup window in the terminal | false | Show messages in a popup window in the terminal. Independent of push. |
| First line of every message, e.g. 'VPS 1 - Scalper' | *(empty)* | Freely selectable text on the first line of every message. |
| Collect messages for X sec. and bundle them (0 = individually) | 5 | Combines messages from this time span into a single push message. 0 = send every message individually and immediately. |

### Structure of the push message

The header line **Reason:** is always present and names the most urgent category. It shows at a glance what the message is about.

```text
DH Drawdown Limiter
------------
Reason: Notice

Lock released - monitoring continues
```

**Several bundled messages** additionally carry their own label on each line.

```text
DH Drawdown Limiter
------------
Reason: Rollover window

- Pause: Rollover window active - reaction and messages suspended
- Connection: Connection disrupted
```

**The 250-character budget** from MetaQuotes applies per notification. Lines are ordered by urgency, and whatever no longer fits is appended as "(+N more, see journal)". When a trigger event coincides with closing, three long messages come together (trigger, confirmation, notice), and the third one no longer fits:

```text
DH Drawdown Limiter
------------
Reason: Loss limit reached

- Loss limit reached: loss 152.40 of 150.00 USD - everything will be closed
- Closing completed: 3 position(s) and 1 pending closed
(+1 more, see journal)
```

Everything is always in the journal in full. If every message is to arrive individually and in full as a push notification, **Collect messages for X sec.** is set to `0`. Each message then goes out individually, with its own header line and its own full budget.

Account values arrive as a **separate, second** message, so they don't compete for the same budget, see **Messages: Content**.

## Messages: Which Events

These switches select the **reasons**. A deselected reason stays silent on every channel. The journal is unaffected and keeps logging everything.

| Variable | Default Value | Description |
|---|---|---|
| Reaction triggered (loss limit / margin / equity level) | true | Send a message when a reaction is triggered (loss limit, margin, or equity level, switched together). |
| Confirmation after the reaction completes | true | Confirmation message after the reaction completes, stating whether all positions and pending orders were closed correctly (relevant if "Collect messages for X sec." is set below 1 second: if closing takes longer than the configured time, further messages are sent). |
| Warning threshold reached | true | When the warning threshold is reached. |
| All-clear reached | true | When the all-clear is reached. |
| Algo trading activated / deactivated | true | When algo trading is manually activated or deactivated. |
| Connection disrupted / restored | true | When the connection is disrupted or restored. |
| Pause turned on or off | true | When the pause is turned on or off in the dashboard. |
| All positions closed | true | When the account has no open positions or pending orders left, i.e. when at least one open position or pending order existed before closing and all of them were fully closed afterward, regardless of who closed them. |
| No push/pop-up at the start and end of the rollover filter | false | `true` silences the two messages at the start and end of the rollover window: no push, no popup. The two journal lines remain. A separate switch, because both messages share the same event class as the **manual** pause and could only be turned off together with it via "Pause turned on or off". |

## Messages: Content

These switches select the **account values** included with the additional message.

| Variable | Default Value | Description |
|---|---|---|
| Send account data? | true | Send an additional message with the account values after an event. |
| 1. Account number | true | The account number. |
| 2. Profit | true | Current open profit/loss. |
| 3. Equity | true | Current equity. |
| 4. Balance | true | Current account balance. |
| 5. Margin level | true | Current margin level. |
| 6. Open positions and pending orders | true | Still-open positions and pending orders. |
| 7. Server time | true | Server time. |

## Dashboard

| Variable | Default Value | Description |
|---|---|---|
| Panel/message language | English | Language used in the dashboard, as well as in the journal and message texts (English or German). |
| Show panel | true | Show the dashboard on the chart. |
| Dashboard label (empty = DH DRAWDOWN LIMITER) | *(empty)* | Your own heading in the dashboard header, for example to tell several accounts apart. Leave empty for "DH DRAWDOWN LIMITER". The text appears as entered. If it is too long it is shortened with "...", and the dashboard does not get wider. Emojis and rare special characters are not in the font and appear as "?". |
| Distance from left (px) | 20 | Distance from the left chart edge. |
| Distance from top (px) | 20 | Distance from the top chart edge. |
| Scale of the whole panel (0.5 to 3.0) | 1.2 | Scales the whole dashboard. |
| Panel width factor (0.6 to 2.0, 1.0 = fit content) | 0.8 | Changes the width of the dashboard. |
| Show a close button per breakdown row | true | Show a dedicated button for closing positions on each row of the position table. |
| Show tooltips on hover | true | Show explanations (tooltip) when the mouse hovers over an element. |
| Show the clock in the panel header | true | Show the time in the top-right of the dashboard header (server time of the latest quote). |
| Show the UTC offset next to the clock | true | Show the UTC time offset from server time next to the dashboard clock. |
| Show timestamp and ping in the status card | true | Display the two additional pieces of information in the status card (on the left, the time of the last algo-trading switch; on the right, the ping). |
| Show percent thresholds below the scale | true | In the locked view, the set threshold values appear beneath their markers on the respective scale. This switch applies only to values displayed as **percentages**. |
| Show money thresholds below the scale | true | In the locked view, the set threshold values appear beneath their markers on the respective scale. This switch applies only to values displayed as a **money amount**. |
| Locked: equity marks show the account level, not the distance | false | Below the equity scale, the account balance that triggers is shown instead of the configured distance. With a calculation basis of 1,500 and a distance of 500, that is 1,000. The tooltip then names both numbers. The value field in the unlocked view stays on the distance, because that is where you type. |
| Locked: loss marks show the amount in account currency | false | Below the loss scale, the loss amount that triggers is shown instead of the percentage. With a balance of 10,000 and 8%, that is 800. If the loss limit is set as a money amount anyway, this switch has no effect. |

The dashboard can be freely moved across the chart (hold the left mouse button down on the dashboard header and drag).

### Names for magic numbers

The names are maintained in the dashboard: the button to the left of the collapse button in the dashboard header switches the lower area to the name list, and a second press switches back. Four tabs choose which numbers appear: those with an open position, with a pending order, all of them together, or only the ones from the trade history. In the bottom row you add a number by hand that appears nowhere yet: the number on the left, the name on the right. Every entry applies immediately, and clearing a field removes the name again.

The names are stored in `MQL5\Files\Drawdown Limiter\MagicNames.txt`, one mapping per line in the format `number=name`. The file is readable and editable by hand and is retained across restarts and updates.

The view does not exist in the strategy tester: an input field accepts no input there, and the test agent's file is empty.

## State

| Variable | Default Value | Description |
|---|---|---|
| Prefix of the state variables | DDLS_ | Identifier under which the Drawdown Limiter stores its state in the global variables. Two instances on different charts must not use the same identifier. |
| Discard saved state at startup | false | Discards the saved state at startup. |

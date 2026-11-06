# show-me-what-you-got (Horizon port, slice 1)

Native Rust port of [show-me-what-you-got](https://github.com/apace100/show-me-what-you-got)
(MIT, (c) 2022 apace100, Modrinth: Show Me What You Got) to the
[Horizon](https://playhorizon.dev) client, ported 1:1.

## Slice 1 (this repo)

The full mod core, 1:1 from upstream at commit `0ee8852`:

- **Item model**: `ItemStack` (id/count/tag), NBT compound subset with SNBT
  parse/print (vanilla `StringNbtReader` semantics: booleans as bytes, suffix
  typing, no-EOF parseCompound) and binary NBT read/write for packets.
- **Core flow**: the `[[smwyg:...]]` tag regex (greedy, line-bounded, Java
  backtracking semantics), `hasSmwygItem` / `extractItem` (malformed tag keeps
  the span with a null stack), and `decorateMessage` (span replaced by the
  item's hoverable text).
- **Per-sender store**: `MessageItemList` (take-on-read `get`, cleared on
  server start).
- **Packet + receiver**: varint start/end + compound NBT, client send path,
  server receiver that enqueues the match through the server's execute queue.
- **Shared text field**: the full `ItemSharingTextFieldWidget` state machine
  from TextFieldWidgetMixin — cursor/selection clamping around the inserted
  item, write-over and erase tracking, suggestion-offset tracking, stale-link
  placeholder (`[stale link]`), and history-recall re-adoption
  (`handle_item_setting`).
- **Chat screen**: send flow (packet for the span, `[[smwyg:...]]`-tagged text
  into the history, no duplicate untagged history entry), init wiring for the
  shared item, history-navigation reset.
- **Client share gate**: the START_CLIENT_TICK logic (left-ctrl + chat key
  edge detection, latch set before the open-chat checks, cursor-empty and
  focused-slot guards).

22 unit tests cover the whole core.

## Game-side seams (tracked)

The render/GUI half needs hooks Horizon does not have yet:

- Tooltip composition in `DrawContext.drawHoverEvent`/`drawTooltip`: custom
  tooltip components (`ItemStackTooltipComponent` renders the hovered stack
  beside the tooltip, honoring the `smwyg:hide_stack` NBT flag;
  `HorizontalLayoutTooltipComponent` is the generic horizontal stacker).
- Chat suggestion completion callback (`ChatInputSuggestor.SuggestionWindow`)
  so the insertion index tracks suggestion inserts.
- Focused-slot and cursor-stack access on container screens, plus per-key
  input polling, for the share gate.
- Chat HUD history writes.

## License

MIT, verbatim from upstream (see LICENSE).

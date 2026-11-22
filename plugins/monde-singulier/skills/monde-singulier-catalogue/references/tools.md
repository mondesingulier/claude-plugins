# The dashboard connector's tools

Connector « Monde Singulier dashboard ». Every write is a dry run unless `apply: true`. Rights: R = `catalogue:read` (designers: `designers:read`), W = `catalogue:write`, P = `catalogue:publish`. A tool the person lacks the right for is not listed for her. Nothing here ever writes to Shopify: the sync is the only road to the store (the two store readers only read).

## Read (no change, no confirmation)

| Tool | What it gives |
|---|---|
| `ms_whoami` | Who the connector acts as: person, teams, role, rights |
| `search_pieces` | Up to 50 pieces by words of title, designer, collection, handle, SKU; filters status, line (`MONDE_SINGULIER`, `MS_EDITIONS`, `THE_VESTIGE`), collection, designer, sync state; lowest price EUR/USD/GBP |
| `get_piece` | The whole piece as its form shows it: English texts, EUR price excl. VAT (USD/GBP for information), dimensions, materials, options, variants with images, free tags (`extraTags`), channels, sync state, and the `token` to send back |
| `get_schema` | Closed lists (categories, option names `mainOptions`, material families, filters, statuses, lines, collection types, store channels, recognitions) and input rules (lengths, the 30 % price rule, what an image must be). Read before creating or changing a piece |
| `list_collections`, `get_collection` | Collections with line, designer, published; one collection with launch day and its `version` |
| `list_designers`, `get_designer` | Designers, dealers, internal brands; one designer and its `version` (never their terms) |
| `search_dealers` (W) | Vestige dealers by name, with site, platform, pickup addresses, piece count (never their contact). Always before `create_dealer` |
| `search_materials` | Materials with family and usage |
| `get_french_texts` (W) | French texts of a piece (`kind: "product"`), designer (`"designer"`) or collection (`"collection"`) beside the English, each with its key and status: automatic, hand-written, missing. The title of a piece has no key (not translated); an SEO key exists only once the English SEO text exists |
| `get_sync_status` | One piece's sync (queued, sent, failed and why, differences) or the sync as a whole |
| `list_drifts` | Open differences with the store, newest first |
| `explain_store_state` | Why a piece is absent from the store or differs (status, collection, designer, channels, what is missing, queue, last error) |
| `list_store_collections` | The store's collections the dashboard does not link, with `counts` (`dashboardFed`, `unmanaged`, `orphanReservedTag`). By default only the `unmanaged` ones (free tag, manual list, other column: the real extra collections); `kind: "dashboard-fed"` pages through the healthy ones (rules on tags the dashboard itself produces, such as `cat:Seating`). Each has handle, title, automatic or manual, rules, product count, `source` and `orphanReservedTag`. `orphanReservedTag: true` = a rule on a reserved tag the dashboard does NOT produce: that collection will drift, it is the signal to watch. `orphanReservedTagOnly: true` filters on it; `offset` / `limit` (50 by default, 200 max). Before creating an extra collection in Shopify |
| `store_drift_report` | Store-side night report: `orphanReservedTagCollections` (with `collectionCounts`; `null` with the reason when the store cannot be read), `bornInShopify` (store products not linked to the dashboard, waiting for a person), `staleOpenAlerts` (open alerts about something already settled, with `howToClose`; `adoptedUnderReview` counts open lines of adopted store pieces still under review: NOT stale, never close them, they protect those pieces' images) |
| `upload_images` (W) | Shows the person a drop box in the chat (JPG, PNG, WEBP; 20 MB a photo, 3 MB for a collection or designer image); gives upload ids. Only in Claude clients that show MCP Apps pages |

## Write (dry run → show → yes → apply)

| Tool | What it does | Right and guard |
|---|---|---|
| `create_piece` | New piece in **Draft** in a collection, with options from the closed list (each combination = a variant) and default channels unless `fields.channels`; Vestige via `vintage` with its dealer (`supplierId`), then pickup, margin scheme and SKU come by themselves | W |
| `update_piece` | Fields: English title, description, SEO, category, price, show price, infos, dimensions, COM, year, free tags `extraTags` (the list sent replaces the piece's free tags; reserved tags refused), filters, channels, vintage | W; price jump > 30 % needs `confirmPriceJumps` |
| `edit_options` | Several operations in order: `rename_option`, `rename_value` (label only, no variant lost), `set_colour` (swatch of a Colour value, dashboard only), `add_option` (existing variants take its first value). After a `rename_option`, later operations use the NEW name | W |
| `remove_option_value` | Removes one value; its variants are archived (any history, photos included) or deleted; the dry run says which | W; `confirm: true` |
| `remove_option` | Removes an option once it has one value left, never the last option | W; `confirm: true` |
| `order_option_values` | Order of an option's values in the dashboard. **The store does not follow it yet** (the answer says so) | W |
| `set_variant` | One variant: SKU, stock, sold out, published, own price, show price, made to order, materials, customs code, dimensions, filters | W; same 30 % rule |
| `add_variant` | One more combination as a variant (a new value is added to its option) | W |
| `remove_variant` | Removes a variant; one ever sold, quoted or carted is archived, never deleted | W; `confirm: true` |
| `set_price` | EUR price excl. VAT of the piece or one variant; USD/GBP follow | W; `confirmPriceJumps` over 30 % |
| `assign_collection` | Moves the piece to another collection (a Vestige piece stays in The Vestige) | W; P if it changes what the site shows |
| `create_material` | New material with its family (an existing name gives it back) | W |
| `prepare_image_uploads` | With a terminal: one signed upload per file (name, type, exact size); PUT within five minutes, then use the upload id within the hour | W |
| `add_piece_images` | Adds images to one variant from `{url}` (public HTTPS address of the file, no login) or `{upload}` (an upload id, usable once). JPG/PNG/WEBP, 20 MB max, 600 × 600 px min, converted and reduced by the server; at most 20 images a call and 5 heavy ones (over 3 MB); 20 images per variant. `position` 1 = main image. An address already on the variant comes back under `alreadyPresent`, so a replay adds nothing | W; token |
| `order_variant_images` | New order of one variant's images; first = main image (the store card) | W |
| `set_hover_image` | The image the variant's card shows on hover (one of its own, never the first); `imageId: null` = the second image | W; token |
| `describe_image` | Alt text (300 characters; `everywhere` for every image showing the same file) | W |
| `set_image_role` | `PRODUCT` or `AMBIANCE` | W |
| `delete_image` | Removes an image | W; `confirm: true`; refused if used elsewhere or it would leave a published piece without a photo |
| `set_french_texts` | Writes French texts by key; they become hand-written and the machine leaves them alone. A French SEO title is refused while the English one is empty | W |
| `hand_back_french_texts` | Gives hand-written French texts back to the machine (retranslated from the English, overwriting what was typed) | W |
| `save_collection` | Create or change a collection: name, description, designer, made in (`location`), type, SEO, `launchDay`, `published`, image (upload id or address) | W; publishing needs P; unpublishing sends back `leaving`; name, designer, type, launch day or publication resend its pieces (100 max) |
| `save_designer` | Create or change a designer: name, bio, type, recognitions, countries, Instagram, SEO, `published`, image | W; publishing needs P; same `leaving` and resend rules; terms never here |
| `create_dealer` | New Vestige dealer with its address (= pickup) and platform (`SHOPIFY`, `WOOCOMMERCE`, `ONEDIBS`, `MANUAL`); the dry run lists lookalikes | P; `notADuplicate: true` once the person said none is it |
| `set_piece_dealer` | Puts a Vestige piece under a dealer or moves it | P |
| `set_visibility` | Publish (`visible`) or unpublish (`hidden`) one piece | P; refused with why (collection/designer not published, no channel, something missing); unpublishing sends back `leaving` |
| `publish_pieces` | Publishes a batch (25 max): without `apply` the summary and one confirmation code; with `apply: true` and the code, the pieces that pass go online, the others are refused with why | P; code valid 15 minutes, for these pieces, if none is saved meanwhile; refused while the store is on emergency stop |
| `request_publication` | For someone without P: asks Partnership on Slack to publish or hide a piece; changes nothing | W |
| `set_new_in` | Puts a published piece in New in (`on: true`, optional `position`, 1 = first) or takes it out | P |
| `resend_piece` | Puts a piece whose last send failed back in the sync queue | P |
| `resolve_conflict` | Marks a difference that is not a conflict as reviewed, with a reason. A real conflict is chosen on Catalogue › Shopify sync | P |
| `set_net_price`, `set_production_cost` | Unit costs; listed only for people with the cost right. Never repeat a cost or margin to anyone else | cost right |

## Not in the connector (by design)

Deleting a piece, collection or designer; merging, renaming or deleting materials; designer terms; orders, customers, payments, invoices, VAT, HubSpot, transport, Settings, rights; bulk import or export; any write to Shopify.

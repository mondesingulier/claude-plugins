# The dashboard connector's tools

Connector « Monde Singulier dashboard » (https://dashboard.monde-singulier.com/mcp). 32 tools. Every write is a dry run unless `apply: true`. Rights: R = `catalogue:read` (designers: `designers:read`), W = `catalogue:write`, P = `catalogue:publish`. Nothing here ever calls Shopify: the sync is the only road to the store.

## Read (no change, no confirmation)

| Tool | What it gives |
|---|---|
| `ms_whoami` | Who the connector acts as: person, teams, role, rights |
| `search_pieces` | Up to 50 pieces by words of title, designer, collection, handle, SKU; filters status (`draft`, `visible`, `hidden`, `removed`), line (`MONDE_SINGULIER`, `MS_EDITIONS`, `THE_VESTIGE`), collection, designer, sync state |
| `get_piece` | The whole piece as its form shows it: texts, EUR price excl. VAT (USD/GBP for information), dimensions, materials, options, variants with images, sync state, and the `token` to send back |
| `get_schema` | Closed lists (categories, options, material families, filters, statuses, lines, collection types, channels, recognitions) and input rules (lengths, the 30 % price rule). Read before creating or changing a piece |
| `list_collections`, `get_collection` | Collections with line, designer, published; one collection with launch day and its `version` |
| `list_designers`, `get_designer` | Designers, dealers, internal brands; one designer and its `version` (never their terms) |
| `search_materials` | Materials with family and usage |
| `get_sync_status` | One piece's sync (queued, sent, failed and why, differences) or the sync as a whole |
| `list_drifts` | Open differences with the store, newest first |
| `explain_store_state` | Why a piece is absent from the store or differs |

## Write (dry run → show → yes → apply)

| Tool | What it does | Right and guard |
|---|---|---|
| `create_piece` | New piece in **Draft** in a collection, with options (each combination = a variant); Vestige via the `vintage` block | W |
| `update_piece` | Fields of a piece (title, description, category, price, show price, infos, SEO, dimensions, COM, year, tags, filters, channels, vintage) | W; price jump > 30 % needs `confirmPriceJumps` |
| `set_variant` | One variant: SKU, stock, sold out, published, own price, materials, customs code, dimensions, filters | W; same 30 % rule |
| `add_variant` | One more combination as a variant | W |
| `remove_variant` | Removes a variant; a variant ever sold, quoted or carted is archived, never deleted | W; `confirm: true` after asking |
| `set_price` | EUR price excl. VAT of the piece or one variant; USD/GBP follow | W; `confirmPriceJumps` over 30 % |
| `assign_collection` | Moves the piece to another collection (a Vestige piece stays in The Vestige) | W; P if it changes what the site shows |
| `create_material` | New material with its family (an existing name gives it back) | W |
| `order_variant_images` | New order of one variant's images; first = main image | W |
| `describe_image` | Alt text (300 characters) | W |
| `set_image_role` | `PRODUCT` or `AMBIANCE` | W |
| `delete_image` | Removes an image | W; `confirm: true`; refused if used elsewhere or it would leave a published piece without a photo |
| `add_piece_images` | Not available yet: add images on the piece's Media in the dashboard | — |
| `save_collection` | Create or change a collection: name, description, designer, made in (`location`), type, SEO, `launchDay`, `published` | W; publishing needs P; unpublishing sends back `leaving`; a change of name, designer, type, launch day or publication resends its pieces (100 max) |
| `save_designer` | Create or change a designer: name, bio, type, recognitions, countries, Instagram, SEO, `published` | W; publishing needs P; same `leaving` and resend rules; terms never here |
| `set_visibility` | Publish (`visible`) or unpublish (`hidden`) a piece | P; refused with the reason if collection/designer not published or something missing; unpublishing sends back `leaving` |
| `request_publication` | For someone without P: asks Partnership on Slack to publish or hide a piece; changes nothing | W |
| `set_new_in` | Puts a published piece in New in (`on: true`) or takes it out (`on: false`) | P |
| `resend_piece` | Puts a piece whose last send failed back in the sync queue | P |
| `resolve_conflict` | Marks a difference that is not a conflict as reviewed, with a reason. A real conflict is chosen on Catalogue › Shopify sync | P |

## Not in the connector (by design)

Deleting a piece, collection or designer; merging or deleting materials; costs, margins, purchase prices, designer terms; orders, customers, payments, invoices, VAT, HubSpot, transport, Settings, rights. Ordering the New in and choosing the hover image are done on dashboard screens.

# CAPS Parser

A small Vue 3 + Vuetify tool for inspecting Pershing CAPS records. Paste lines from a CAPS file and each fixed-width field is split out and labeled, with its start/end column shown. Signed-overpunch numbers (quantity, price, fees, commissions, etc.) are decoded into readable values.

It handles record types 1, 2, 3 and 5. Lines with any other record type, such as the rarely used R4, are ignored.

## Getting started

Requires [Node.js](https://nodejs.org/) 20.19 or newer.

```sh
git clone https://github.com/uclnj/caps-parser.git
cd caps-parser
npm install
npm run dev
```

Then open http://localhost:5173 in your browser.

## Usage

1. Open a CAPS file in a text editor.
2. Copy the four lines that make up one trade: its 1, 2, 3 and 5 records.
3. Paste them into the **Source Record** box.

The fields below the box fill in as you paste or type. Each label shows the field name and its column range in the record, e.g. `Account [22/31]`.

## Building

```sh
npm run build     # outputs to dist/
npm run preview   # serves the built files locally
```

<!-- @format -->

<template>
  <!-- lead container -->
  <v-container fluid>

    <v-row dense>
      <v-col cols="12">
        <v-textarea v-model="input" variant="outlined" density="compact" class="bg-lightbeige" hide-details
          @update:modelValue="parseAll" label="Source Record" style="font-family: 'Courier New', Courier, monospace;"
          columns="134" rows="4" maxlength="535" no-resize></v-textarea>
      </v-col>
    </v-row>

    <v-row dense>
      <v-col cols="12">
        <v-card flat>
          <v-card-text>
            <div class="d-flex align-center flex-wrap">
              <template v-for="rec in records" :key="rec.id">
                <v-sheet v-for="(f, x) in rec.fields" :key="`${rec.id}-${x}`" class="ma-1"
                  :style="`width:${f.length * rec.charWidth}px;min-width:150px;`">
                  <v-text-field :model-value="f.output" readonly variant="outlined" density="compact" hide-details
                    :class="f.color || ''" style="font-family: 'Courier New', Courier, monospace;"
                    :label="`${f.label} [${rec.totals[x] - f.length}/${rec.totals[x] - 1}]`"></v-text-field>
                </v-sheet>
              </template>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

function runningTotals(fields) {
  let pos = 1
  return fields.map((f) => (pos += f.length))
}

export default {
  name: "caps-parser",
  computed: {
    // Record types in display order. R4 is in the CAPS spec but almost never
    // appears; to support it, add an R4 field list to data and an entry here.
    records() {
      return [
        { id: '1', fields: this.R1, charWidth: 21 },
        { id: '2', fields: this.R2, charWidth: 22 },
        { id: '3', fields: this.R3, charWidth: 22 },
        { id: '5', fields: this.R5, charWidth: 22 },
      ].map((r) => ({ ...r, totals: runningTotals(r.fields) }))
    },
  },
  methods: {
    // Pershing signed-overpunch number: the last character encodes both the final
    // digit and the sign. `period` is the count of integer digits.
    // Returns '' for incomplete or malformed input (e.g. while typing).
    pershing_string_to_number(source, period) {
      const fmt = this.PershingFractionalNumberFormat.find((e) => e.digit === source.slice(-1))
      if (!fmt) return ''
      const digits = source.slice(0, -1)
      // Keep the decimal part as a string so its leading zeros survive.
      const n = +`${digits.slice(0, period)}.${digits.slice(period)}${fmt.value}`
      if (Number.isNaN(n)) return ''
      return fmt.sign ? -n : n
    },
    parse(input, fields) {
      let pos = 0
      for (const f of fields) {
        const value = input.slice(pos, pos + f.length)
        pos += f.length
        f.output = f.pnum ? this.pershing_string_to_number(value, f.period) : value.trim()
      }
    },
    parseAll() {
      for (const line of this.input.split('\n')) {
        const rec = this.records.find((r) => r.id === line[3])
        if (rec) this.parse(line, rec.fields)
      }
    },
  },
  data: () => ({
    input: '',
    PershingFractionalNumberFormat: [
      { digit: '{', value: 0, sign: false },
      { digit: 'A', value: 1, sign: false },
      { digit: 'B', value: 2, sign: false },
      { digit: 'C', value: 3, sign: false },
      { digit: 'D', value: 4, sign: false },
      { digit: 'E', value: 5, sign: false },
      { digit: 'F', value: 6, sign: false },
      { digit: 'G', value: 7, sign: false },
      { digit: 'H', value: 8, sign: false },
      { digit: 'I', value: 9, sign: false },
      { digit: '}', value: 0, sign: true },
      { digit: 'J', value: 1, sign: true },
      { digit: 'K', value: 2, sign: true },
      { digit: 'L', value: 3, sign: true },
      { digit: 'M', value: 4, sign: true },
      { digit: 'N', value: 5, sign: true },
      { digit: 'O', value: 6, sign: true },
      { digit: 'P', value: 7, sign: true },
      { digit: 'Q', value: 8, sign: true },
      { digit: 'R', value: 9, sign: true },
    ],
    R1: [
      { field: 'ibd_num', length: 3, output: '', label: 'IBD Number', color: 'bg-blue-lighten-5' },
      { field: 'record_id', length: 1, output: '', label: 'Record ID' },
      { field: 'trade_date', length: 8, output: '', label: 'Trade Date' },
      { field: 'trade_ref_num', length: 6, output: '', label: 'Trade Ref' },
      { field: 'pershing_office_num', length: 3, output: '', label: 'Office' },
      { field: 'account_number', length: 10, output: '', label: 'Account' },
      { field: 'account_name', length: 32, output: '', label: 'Name' },
      { field: 'buy_sell_indicator', length: 1, output: '', label: 'Buy/Sell' },
      { field: 'cancel_indicator', length: 1, output: '', label: 'Cancel' },
      { field: 'caps_source', length: 3, output: '', label: 'Source' },
      { field: 'trade_quantity', length: 13, output: '', label: 'Quantity', pnum: true, period: 9, color: 'bg-brown-lighten-5' },
      { field: 'price', length: 13, output: '', label: 'Price', pnum: true, period: 6, color: 'bg-brown-lighten-5' },
      { field: 'product_code', length: 1, output: '', label: 'Code' },
      { field: 'cusip_num', length: 12, output: '', label: 'CUSIP' },
      { field: 'short_name', length: 10, output: '', label: 'Short' },
      { field: 'order_type', length: 1, output: '', label: 'Order' },
      { field: 'misc_fee', length: 13, output: '', label: 'Fee', pnum: true, period: 11, log: true, color: 'bg-red-lighten-5' },
      // { field: 'not_used', length: 2, output: '', label: 'Not Used' },
    ],
    R2: [
      { field: 'ibd_num', length: 3, output: '', label: 'IBD Number', color: 'bg-blue-lighten-5' },
      { field: 'record_id', length: 1, output: '', label: 'Record ID' },
      { field: 'market_code', length: 1, output: '', label: 'Market' },
      { field: 'blotter_code', length: 1, output: '', label: 'Blotter' },
      { field: 'security_type', length: 1, output: '', label: 'Type' },
      { field: 'stock_symbol', length: 9, output: '', label: 'Symbol' },
      { field: 'security_description', length: 40, output: '', label: 'Desc' },
      { field: 'cents_per_share', length: 9, output: '', label: 'Cents/Share' },
      { field: 'discount_percent', length: 7, output: '', label: 'Discount', pnum: true, period: 5 },
      { field: 'paycode', length: 2, output: '', label: 'Paycode' },
      { field: 'master_client_mnemonic', length: 9, output: '', label: 'Master' },
      { field: 'institutional_retail_indicator', length: 1, output: '', label: 'IRI' },
      { field: 'state_code', length: 3, output: '', label: 'State' },
      { field: 'commission_indicator', length: 3, output: '', label: 'COM' },
      { field: 'firm_trading_account', length: 9, output: '', label: 'Firm' },
      { field: 'posted_date', length: 8, output: '', label: 'Posted' },
      { field: 'settlement_date', length: 8, output: '', label: 'Settle' },
      { field: 'pershing_internal_use', length: 17, output: '', label: 'PII' },
      { field: 'recycle_indicator', length: 1, output: '', label: 'Recycle' },
    ],
    R3: [
      { field: 'ibd_number', length: 3, output: '', label: 'IBD', color: 'bg-blue-lighten-5' },
      { field: 'record_id', length: 1, output: '', label: 'Record ID' },
      { field: 'as_of_trade_indicator', length: 1, output: '', label: 'AS/OF' },
      { field: 'distribution_indicator', length: 1, output: '', label: 'Dist' },
      { field: 'explode_indicator', length: 1, output: '', label: 'Explode' },
      { field: 'revenue_type', length: 1, output: '', label: 'Revenue Type' },
      { field: 'investment_pro_ip_number', length: 3, output: '', label: 'IP#' },
      { field: 'inv_pro_caps_revenue_center', length: 3, output: '', label: 'CENTER' },
      { field: 'inv_pro_caps_office_number', length: 3, output: '', label: 'OFF' },
      { field: 'inv_pro_split_percent', length: 6, output: '', label: 'SP%', pnum: true, period: 3 },
      { field: 'total_amount_of_revenue', length: 13, output: '', label: 'Total', pnum: true, period: 11, color: 'bg-green-lighten-5' },
      { field: 'pershing_charge', length: 11, output: '', label: 'Pershing Charge', pnum: true, period: 9, color: 'bg-red-lighten-5' },
      { field: 'legacy_from_inv_pro_number', length: 3, output: '', label: 'Legacy' },
      { field: 'caps_revenue_associated_with_the_from', length: 3, output: '', label: 'RevFrom' },
      { field: 'caps_office_number_associated_with_the_from', length: 3, output: '', label: 'OffFrom' },
      { field: 'total_order_commission', length: 13, output: '', label: 'Order Comm', pnum: true, period: 11, color: 'bg-red-lighten-5' },
      { field: 'spread_straddle_mutual_fund_values', length: 1, output: '', label: 'Spread' },
      { field: 'correction_code', length: 1, output: '', label: 'CCode' },
      { field: 'security_modifier', length: 1, output: '', label: 'SecMod' },
      { field: 'security_calculation_code', length: 1, output: '', label: 'SecCalc' },
      { field: 'principal', length: 13, output: '', label: 'Principle', color: 'bg-brown-lighten-5', pnum: true, period: 11 },
      { field: 'crd_number', length: 7, output: '', label: 'CRD' },
      { field: 'source_of_input', length: 2, output: '', label: 'Source' },
      { field: 'for_pershing_internal_use_only', length: 25, output: '', label: 'PII' },
      { field: 'service_charge', length: 13, output: '', label: 'Charge', pnum: true, period: 11 },
    ],
    R5: [
      { field: 'ibd_number', length: 3, output: '', label: 'IBD' },
      { field: 'record_id', length: 1, output: '', label: 'Record ID' },
      { field: 'option_root_id', length: 6, output: '', label: 'Root ID' },
      { field: 'expiration_date', length: 6, output: '', label: 'Expire Date' },
      { field: 'call_put_indicator', length: 1, output: '', label: 'Call/Put' },
      { field: 'strike_price', length: 8, output: '', label: 'Strike Price', pnum: true, period: 5 },
      { field: 'master_client_name', length: 32, output: '', label: 'Client' },
      { field: 'internal_use_only', length: 76, output: '', label: 'IUU' },
      //{ field: 'pershing_internal_order_reference_number', length: 4, output: '', label: 'Ref#' },
      //{ field: 'expanded_ip_number', length: 4, output: '', label: 'IP' },
    ]
  }),
}
</script>

<style scoped></style>
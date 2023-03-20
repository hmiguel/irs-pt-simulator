<template>
  <div class="container">
    <table>
      <thead>
        <tr>
          <th></th>
          <th>Descrição</th>
          <th>Valor</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in rows" :key="index" :class="{ 'bold': boldRows.includes(row) }">
          <td>{{ index + 1 }}</td>
          <td>{{ row.description }}</td>
          <td>
            <input v-if="row.input" v-model="row.value" type="number" />
            <span v-else>{{ row.value }} €</span>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="total">
      <span>Valor a reembolsar {{ payReturnTotal }} €</span>
    </div>
    <div class="totalTax">
      <span>Taxa Efetiva de Tributação - {{ taxEffectiveTotal*100 }} %</span>
    </div>
  </div>
</template>
<script>



export default {
  data() {
    return {
      rows: [
        { description: "RENDIMENTO GLOBAL", input: true, value: 0, bold: true },
        { description: "Deduções específicas", input: false, value: 0 },
        { description: "Perdas a recuperar", input: false, value: 0 },
        { description: "Abatimentos", input: false, value: 0 },
        { description: "Deduções ao rendimento", input: false, value: 0 },
        { description: "RENDIMENTO COLETÁVEL [1-(2+3+4+5)]", input: false, value: 0, bold: true },
        { description: "Quociente rendimentos anos anteriores", input: false, value: 0 },
        { description: "Rendimentos isentos englobados para determinação da taxa", input: false, value: 0 },
        { description: "TOTAL DO RENDIMENTO PARA DETERMINAÇÃO DA TAXA (6+8-7)", input: false, value: 0, bold: true },
        { description: "Coeficiente familiar 1,00 ; taxa 28,500%", input: false, value: 0 },
        { description: "IMPORTÂNCIA APURADA (9 : COEF x TAXA)", input: false, value: 0, bold: true },
        { description: "Parcela a Abater", input: false, value: 0 },
        { description: "Imposto correspondente a rendimentos anos anteriores", input: false, value: 0 },
        { description: "Imposto correspondente a rendimentos isentos", input: false, value: 0 },
        { description: "Taxa adicional (0,00 x 0,0% + 0,00 x 0%) x 1,00", input: false, value: 0 },
        { description: "Excesso em relação ao limite do quociente familiar", input: false, value: 0 },
        { description: "Imposto relativo a tributações autónomas", input: false, value: 0 },
        { description: "COLETA TOTAL [(11-12) x (1,00) + 13 - 14 + 15 + 16+17]", input: false, value: 0, bold: true },
        { description: "Deduções à coleta", input: true, value: 0 },
        { description: "Benefício municipal (0,00% da coleta)", input: false, value: 0 },
        { description: "Acréscimo à coleta", input: false, value: 0 },
        { description: "COLETA LÍQUIDA [ 18 - 19 - 20 (>=0) + 21]", input: false, value: 0, bold: true },
        { description: "Pagamentos por conta", input: false, value: 0 },
        { description: "Retenção na fonte", input: true, value: 0 },
        { description: "IMPOSTOS APURADOS [22 - (23 + 24)]", input: false, value: 0, bold: true },
        { description: "Juros de retenção-poupança", input: false, value: 0 },
        { description: "Sobretaxa-resultado", input: false, value: 0 },
        { description: "Juros compensatórios", input: false, value: 0 },
        { description: "Juros indemnizatórios", input: false, value: 0 },	

      ],
    };
  },
  methods: {
    setSpecificDeductions(ssPercent, specificDeduction) {
      this.rows[1].value =  (Math.max(parseFloat(this.rows[0].value)*ssPercent, specificDeduction)).toFixed(2);
    },
    setCollectableIncome() {
      this.rows[5].value = (parseFloat(this.rows[0].value) - (parseFloat(this.rows[1].value) 
      + parseFloat(this.rows[2].value) + parseFloat(this.rows[3].value) + parseFloat(this.rows[4].value))).toFixed(2);
    },
    setTotalIncomeForTaxDetermination() {
      this.rows[8].value = (parseFloat(this.rows[5].value) + parseFloat(this.rows[7].value) - parseFloat(this.rows[6].value)).toFixed(2);
    },
    // 9 
    getTaxValuesForTotalIncome(){
      for (let i = 1; i <= Object.keys(this.irsTable).length; i++) {
        if (this.rows[5].value < this.irsTable[i].to) {
          return this.irsTable[i];
        }
      }
    },
    // 10 Importância apurada
    setTaXValueDescription() {
      this.familiarQuotient = 1;
      this.rows[9].description = "Coeficiente familiar " + this.familiarQuotient + "; taxa " + this.taxValue.tax*100 + "%";
      this.rows[9].value = "";
    },
    // 11 IMPORTÂNCIA APURADA (9 : COEF x TAXA)
    setTaxValue() {
      this.rows[10].value = (parseFloat(this.rows[8].value) * 1.0 * this.taxValue.tax).toFixed(2);
    },
    // 12 Parcela a abater
    setParcelToDeduct() {
      this.rows[11].value = this.taxValue.parcel; 
    },
    // 13 Imposto correspondente a rendimentos anos anteriores
    setTaxForPreviousYears() {
      this.rows[12].value = 0;
    },
    // 14 Imposto correspondente a rendimentos isentos
    setTaxForExemptIncome() {
      this.rows[13].value = 0;
    },
    // 15 Taxa adicional (0,00 x 0,0% + 0,00 x 0%) x 1,00
    setAdditionalTax() {
      this.rows[14].value = 0;
    },
    // 16 Excesso em relação ao limite do quociente familiar
    setExcess() {
      this.rows[15].value = 0;
    },
    // 17 Imposto relativo a tributações autónomas
    setTaxForAutonomousTaxation() {
      this.rows[16].value = 0;
    },
    // 18 COLETA TOTAL [(11-12)x(1,00)+13-14+15+16+17]
    setTotalCollect() {
      this.rows[17].value = ((this.rows[10].value - this.rows[11].value) * (1.00) + this.rows[12].value - this.rows[13].value + this.rows[14].value + this.rows[15].value + this.rows[16].value).toFixed(2);
    },
    // 20 Benefício Municipal
    setMunicipalBenefit(county) {
      const countTax = parseFloat(this.countiesTaxes[county]);
      this.rows[19].value = (this.rows[17].value * (0.05 - countTax)).toFixed(2);
      this.rows[19].description = "Benefício Municipal (" + countTax*100 + " % da coleta)";
    },
    // 21 Acréscimo à coleta
    setCollectIncrease() {
      this.rows[20].value = 0;
    },
    // 22 COLETA LÍQUIDA [18-19-20(>=0)+21]
    setNetCollectValue() {
      this.rows[21].value = (this.rows[17].value - this.rows[18].value - Math.max(this.rows[19].value, 0) + this.rows[20].value).toFixed(2);
    },
    // 23 Pagamentos por conta
    setPaymentsForAccount() {
      this.rows[22].value = 0;
    },
    // 25 IMPOSTOS APURADOS (22 - (23 + 24))
    setTaxValue25() {
      this.rows[24].value = ((this.rows[22].value + this.rows[23].value) - this.rows[21].value).toFixed(2);
    },
    // 26 Juros de retenção-poupança
    setSavingsRetentionInterest(euribor) {
      this.rows[25].value = (euribor*0.72)*this.rows[24].value;
    },
    // 27 Sobretaxa-resultado
    setOverTax() {
      this.rows[26].value = 0;
    },
    // 28 Juros compensatórios
    setCompensatoryInterest() {
      this.rows[27].value = 0;
    },
    // 29 Juros indemnizatórios
    setIndemnityInterest() {
      this.rows[28].value = 0;
    },
    setPayReturnTotal(){
      this.payReturnTotal = (parseFloat(this.rows[24].value) + parseFloat(this.rows[25].value) + parseFloat(this.rows[26].value) + parseFloat(this.rows[27].value) + parseFloat(this.rows[28].value)).toFixed(2);
    },
    setTaxTotal(){
      this.taxEffectiveTotal = (parseFloat(this.rows[21].value) / parseFloat(this.rows[0].value)).toFixed(4);
    },
    updateComputedValues() {
      const metadata = require("../metadata.json");
      const year = "2022";
      const county = "lisboa";
      const euribor =  metadata[year]["euribor"];
      const ssPercent = metadata[year]["social_security_tax"];
      const specificDeduction = metadata[year]["specific_deduction"];
      this.irsTable = metadata[year]["table"];
      this.countiesTaxes = metadata[year]["county_tax"];
      this.setSpecificDeductions(ssPercent, specificDeduction);   
      this.setCollectableIncome();
      this.setTotalIncomeForTaxDetermination();
      this.taxValue = this.getTaxValuesForTotalIncome(year);
      this.setTaXValueDescription();
      this.setTaxValue();
      this.setParcelToDeduct();
      this.setTaxForPreviousYears();
      this.setTaxForExemptIncome();
      this.setAdditionalTax();
      this.setExcess();
      this.setTaxForAutonomousTaxation();
      this.setTotalCollect();
      this.setMunicipalBenefit(county);
      this.setCollectIncrease();
      this.setNetCollectValue();
      this.setPaymentsForAccount();
      this.setTaxValue25();
      this.setSavingsRetentionInterest(euribor);
      this.setOverTax();
      this.setCompensatoryInterest();
      this.setIndemnityInterest();
      this.setPayReturnTotal();
      this.setTaxTotal();
    },
  },
  computed: {
    boldRows() {
      return this.rows.filter((row) => row.bold);
    },
  },
  watch: {
    rows: {
      deep: true,
      handler: function() {
        this.updateComputedValues();
      },
    },
  },
};
</script>


<style>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

thead {
  background-color: #ddd;
}

th {
  text-align: left;
  padding: 10px;
}

td {
  padding: 10px;
}

input[type="number"] {
  width: 100%;
  padding: 5px;
  border: 1px solid #ddd;
  border-radius: 3px;
}

input[type="number"]:focus {
  outline: none;
  border-color: #333;
}
</style>

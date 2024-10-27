<template>
  <div>
    <div class="search-sort">
      <InputText v-model="searchQuery" placeholder="Search by Country Name" @input="fetchCountries" />
    </div>
    <Card style="width: 25rem; overflow: hidden">
      <!-- DataTable with pagination set to 25 rows per page -->
      <DataTable class="table" :value="countries" paginator :rows="25" :first="first" @page="onPageChange" @row-click="showCountryDetails">
        <Column field="flags.png" header="Flags" >
          <template #body="slotProps">
              <div class="flex flex-wrap gap-2">
                <img :src="slotProps.data.flags.png" :alt="slotProps.data.flags.png" width="50" />
              </div>
          </template>
        </Column>
        <Column field="name.official" header="Country Name" sortable />
        <Column field="cca2" header="2-Character Code" />
        <Column field="cca3" header="3-Character Code" />
        <Column field="name.nativeName.eng.official" header="Native Name" />
        <Column field="altSpellings.0" header="Alt. Names" />
        <Column field="idd.root" header="Calling Codes" :body="callingCodeTemplate" />
      </DataTable>
  </Card>
    <!-- Modal to show details -->
    <Dialog header="Country Details" v-model:visible="showModal" :modal="true" width="600">
      <div v-if="selectedCountry">
        <h3>{{ selectedCountry.name.official }}</h3>
        <img :src="selectedCountry.flags.png" alt="flag" width="100" />

        <p><strong>Common Name:</strong> {{ selectedCountry.name.common }}</p>
        <p><strong>Native Name:</strong> 
          <span v-for="(name, lang) in selectedCountry.name.nativeName" :key="lang">
             <Tag :value="name.official" class="name-tag"></Tag>
          </span>
        </p>
        <p><strong>Country Codes:</strong> {{ selectedCountry.cca2 }} / {{ selectedCountry.cca3 }} / {{ selectedCountry.ccn3 }}</p>
        <p><strong>Alternative Spellings:</strong> {{ selectedCountry.altSpellings.join(', ') }}</p>
        <p><strong>Region:</strong> {{ selectedCountry.region }}</p>
        <p><strong>Subregion:</strong> {{ selectedCountry.subregion }}</p>
        <p><strong>Capital:</strong> {{ selectedCountry.capital?.join(', ') }}</p>
        <p><strong>Population:</strong> {{ selectedCountry.population }}</p>
        <p><strong>Area:</strong> {{ selectedCountry.area }} km²</p>
        <p><strong>Timezone:</strong> {{ selectedCountry.timezones.join(', ') }}</p>
        <p><strong>Continent:</strong> {{ selectedCountry.continents.join(', ') }}</p>
        <p><strong>Flag:</strong> {{ selectedCountry.flag }}</p>
        <p><strong>Languages:</strong>
          <span v-for="(lang, code) in selectedCountry.languages" :key="code">
            <Tag :value="lang" class="name-tag"></Tag>
          </span>
        </p>
        <p><strong>Currency:</strong>
          <span v-for="(currency, code) in selectedCountry.currencies" :key="code">
            <Tag :value="currency.name + ' ' + currency.symbol" class="name-tag"></Tag>
          </span>
        </p>
        <p><strong>Calling Code:</strong> {{ selectedCountry.idd.root }}{{ selectedCountry.idd.suffixes ? selectedCountry.idd.suffixes.join(', ') : '' }}</p>
        <p><strong>Google Maps:</strong> <a :href="selectedCountry.maps.googleMaps" target="_blank">View on Google Maps</a></p>
        <p><strong>OpenStreetMap:</strong> <a :href="selectedCountry.maps.openStreetMaps" target="_blank">View on OpenStreetMap</a></p>
      </div>
    </Dialog>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, defineComponent, h } from 'vue';
import axios from 'axios';
import InputText from 'primevue/inputtext';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Dialog from 'primevue/dialog';
import Dropdown from 'primevue/dropdown';
import Tag from 'primevue/tag';
interface Country {
  flags: { png: string };
  name: {
    official: string;
    common: string;
    nativeName?: Record<string, { official: string; common: string }>;
  };
  cca2: string;
  ccn3: string;
  cca3: string;
  altSpellings: string[];
  idd: { root: string; suffixes?: string[] };
  capital: string[];
  region: string;
  subregion: string;
  population: number;
  area: number;
  timezones: string[];
  continents: string[];
  flag: string;
  languages: Record<string, string>;
  currencies: Record<string, { name: string; symbol: string }>;
  maps: { googleMaps: string; openStreetMaps: string };
}
export default defineComponent({
  components: { InputText, DataTable, Column, Dialog,Tag },
  setup() {
    const countries = ref<Country[]>([]);
    const searchQuery = ref<string>('');
    const sortOrder = ref<string | null>(null);
    const first = ref<number>(0);
    const showModal = ref<boolean>(false);
    const selectedCountry = ref<Country | null>(null);

    const sortOptions = [
      { label: 'Ascending', value: 'asc' },
      { label: 'Descending', value: 'desc' },
    ];

    const fetchCountries = async (): Promise<void> => {
      const searchTerm = searchQuery.value;
      const order = sortOrder.value === 'desc' ? -1 : 1;

      try {
        const response = await axios.get<Country[]>('https://restcountries.com/v3.1/all');
        let result = response.data;

        if (searchTerm) {
          result = result.filter(country =>
            country.name.official.toLowerCase().includes(searchTerm.toLowerCase())
          );
        }

        result.sort((a, b) => (a.name.official > b.name.official ? order : -order));
        countries.value = result;
      } catch (error) {
        console.error(error);
      }
    };

    const onPageChange = (event: { first: number }): void => {
      first.value = event.first;
    };

    const showCountryDetails = (event: { data: Country }): void => {
       selectedCountry.value = event.data;
      showModal.value = true;
    };

    onMounted(fetchCountries);

    return {
      countries,
      searchQuery,
      sortOrder,
      sortOptions,
      fetchCountries,
      first,
      onPageChange,
      showCountryDetails,
      showModal,
      selectedCountry,
    };
  },
  methods: {
    flagTemplate(rowData: Country) {
      return h('img', { src: rowData.flags.png, alt: 'flag', width: '50' });
    },
    callingCodeTemplate(rowData: Country) {
      return rowData.idd.root + (rowData.idd.suffixes || []).join(', ');
    },
  },
});
</script>

<style>
.search-sort {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.table .p-datatable-table-container{
  max-height: 60vh;
  overflow: scroll
}
.name-tag {
  margin-left: 5px;
}
</style>

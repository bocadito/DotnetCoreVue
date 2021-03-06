<template>
  <div class="container-fluid mt-4">
    <h1 class="h1">Acronym Record Editor</h1>
    <b-alert :show="loading" variant="info">Loading...</b-alert>
    <b-row>
      <b-col>
        <table class="table table-striped">
          <thead>
            <tr>
              <th>Name</th>
              <th>Value</th>
              <th>Description</th>
              <th>&nbsp;</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="record in records" :key="record.id">
              <td>{{ record.name | capitalized}}</td>
              <td>{{ record.value }}</td>
              <td>{{ record.description }}</td>
              <td class="text-right">
                <a href="#" @click.prevent="updateAcronymRecord(record)">Edit</a> -
                <a href="#" @click.prevent="deleteAcronymRecord(record.id)">Delete</a>
              </td>
            </tr>
          </tbody>
        </table>
        <acronym-editor-record-table
          :records="records"
          @deleteAcronymRecord="deleteAcronymRecord"></acronym-editor-record-table>
      </b-col>
      <b-col lg="4">
        <b-card :title="(model.id ? 'Edit Acronym ID#' + model.id : 'New Acronym Record')">
          <form @submit.prevent="createAcronymRecord">
            <b-form-group label="Name">
              <b-form-input type="text" v-model="model.name"></b-form-input>
            </b-form-group>
            <b-form-group label="Value">
              <b-form-input rows="4" v-model="model.value"></b-form-input>
            </b-form-group>
            <b-form-group label="Description (optional)">
              <b-form-textarea rows="4" v-model="model.description"></b-form-textarea>
            </b-form-group>
            <div>
              <b-btn type="submit" variant="success">Save Record</b-btn>
            </div>
          </form>
        </b-card>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import api from "@/services/AcronymRecordsApiService";
import AcronymEditorRecordTable from "./AcronymEditorRecordTable";

export default {
  components: {AcronymEditorRecordTable},
  data() {
    return {
      loading: true,
      records: [],
      model: {}
    };
  },
  async created() {
    this.getAll();
  },
  watch: {
    'model.value': {
      handler(newValue, oldValue) {
        console.log(`You changed the record value from ${oldValue} to ${newValue}`)
      },
    },
  },
  filters: {
    capitalized: function(value) {
      if (!value) return '';

      return value.toUpperCase();
    }
  },
  methods: {
    async getAll() {
      this.loading = true;

      try {
        this.records = await api.getAll();
      } finally {
        this.loading = false;
      }
    },
    async updateAcronymRecord(acronymRecord) {
      // We use Object.assign() to create a new (separate) instance
      this.model = Object.assign({}, acronymRecord);
    },
    async createAcronymRecord() {
      const isUpdate = !!this.model.id;

      this.formHasErrors = false;
      if (!this.model.name || this.model.name.indexOf(" ") > 0) {
        this.formHasErrors = true;
        this.$bvToast.toast("Invalid acronym", {
          title: "Missing Data.",
          variant: "info"
        });

        return;
      }
      if (!this.model.value) {
        this.formHasErrors = true;
        this.$bvToast.toast("Invalid acronym value", {
          title: "Missing Data.",
          variant: "info"
        });

        return;
      }

      if (isUpdate) {
        await api.update(this.model.id, this.model);
      } else {
        await api.create(this.model);
      }

      // Clear the data inside of the form
      this.model = {};

      // Fetch all records again to have latest data
      await this.getAll();
    },
    async deleteAcronymRecord(id) {
      if (confirm("Are you sure you want to delete this record?")) {
        // if we are editing a acronym record we deleted, remove it from the form
        if (this.model.id === id) {
          this.model = {};
        }

        await api.delete(id);
        await this.getAll();
      }
    }
  }
};
</script>

<template>
    <div class="documents-wrapper">
        <div class="controls"
            v-if="!disableControls">
            <slot :id="id"
                name="controls"
                :is-mobile="isMobile"
                :type="type"
                :uploadLink="uploadLink"
                :fetch="fetch"
                :internal-query="internalQuery">
                <uploader :params="{ documentable_type: type, documentable_id: id }"
                    :url="uploadLink"
                    multiple
                    @upload-successful="fetch();"/>
                <a class="button has-margin-left-small"
                    @click="fetch()">
                    <span v-if="!isMobile">
                        {{ i18n('Reload') }}
                    </span>
                    <span class="icon">
                        <fa icon="sync"/>
                    </span>
                </a>
                <p class="control has-icons-left has-icons-right has-margin-left-large">
                    <input v-model="internalQuery"
                        class="input is-rounded"
                        type="text"
                        :placeholder="i18n('Filter')">
                    <span class="icon is-small is-left">
                        <fa icon="search"/>
                    </span>
                    <span v-if="internalQuery"
                        class="icon is-small is-right clear-button"
                        @click="internalQuery = ''">
                        <a class="delete is-small"/>
                    </span>
                </p>
            </slot>
        </div>
        <div class="has-margin-top-large"
            :class="{'columns is-mobile is-multiline': !compact}">
            <div v-for="(doc, index) in filteredDocuments"
                :key="doc.id"
                :class="{ 'column is-half-mobile is-one-third-desktop': !compact }">
                <component :is="component"
                    :file="doc.file"
                    @delete="destroy(index)"/>
            </div>
        </div>
    </div>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faPlus, faSync, faSearch } from '@fortawesome/free-solid-svg-icons';
import { mapState } from 'vuex';
import { Uploader } from '@enso-ui/uploader/bulma';
import File from '@core-pages/files/components/File.vue';
import Document from './Document.vue';

library.add(faPlus, faSync, faSearch);

export default {
    name: 'Documents',

    components: { Document, File, Uploader },

    inject: ['errorHandler', 'i18n'],

    props: {
        id: {
            type: [String, Number],
            required: true,
        },
        type: {
            type: String,
            required: true,
        },
        query: {
            type: String,
            default: '',
        },
        compact: {
            type: Boolean,
            default: false,
        },
        disableControls: {
            type: Boolean,
            default: false,
        },
    },

    data: () => ({
        documents: [],
        loading: false,
        internalQuery: '',
    }),

    computed: {
        ...mapState('layout', ['isMobile']),
        filteredDocuments() {
            const query = this.internalQuery.toLowerCase();

            return query
                ? this.documents.filter(({ file }) => file.name.toLowerCase().indexOf(query) > -1)
                : this.documents;
        },
        count() {
            return this.filteredDocuments.length;
        },
        uploadLink() {
            return route('core.documents.store');
        },
        component() {
            return this.compact
                ? 'document'
                : 'file';
        },
    },

    watch: {
        query() {
            this.internalQuery = this.query;
        },
    },

    created() {
        this.fetch();
    },

    methods: {
        fetch() {
            this.loading = true;

            axios.get(route('core.documents.index'), {
                params: { documentable_type: this.type, documentable_id: this.id },
            }).then(({ data }) => {
                this.documents = data;
                this.loading = false;
                this.$emit('update');
            }).catch(this.errorHandler);
        },
        destroy(index) {
            this.loading = true;

            axios.delete(route(
                'core.documents.destroy',
                this.documents[index].id, false,
            )).then(() => {
                this.loading = false;
                this.documents.splice(index, 1);
                this.$emit('update');
            }).catch(this.errorHandler);
        },
    },
};
</script>


<style lang="scss">
    .documents-wrapper .controls {
        display: flex;
        justify-content: center;
    }
</style>

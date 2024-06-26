<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <b-form @submit.prevent="submitDisplayContent">
    <div class="editor">
      <b-card title="Modify display content" class="form_card">
        <b-card-text>
          <b-form-checkbox
            :checked="ignoreSchedule"
            @change="onIgnoreScheduleChange"
            class="form_group mb-4"
          >
            Override content defined in Scheduler
          </b-form-checkbox>
          <b-form-group label="Choose a picture:" class="form_group">
            <b-form-file v-model="image" accept="image/*"></b-form-file>
          </b-form-group>
          <b-form-group
            label="Or choose a template:"
            class="form_group"
            description="Choosing a different template will override existing picture and fields"
          >
            <b-form-select
              v-model="selectedTemplateId"
              value-field="uuid"
              text-field="name"
              :options="templates"
              @input="onSelectedTemplateChange"
            >
            </b-form-select>
          </b-form-group>
          <b-input-group>
            <label for="padding">Header/Footer size (aka padding)</label>
            <b-form-input
              id="padding"
              label=""
              v-model="padding"
              type="number"
              min="0"
            ></b-form-input>
          </b-input-group>
          <label for="rooms"
            >This template can support maximal <strong>{{ maxRooms }}</strong>
            rooms
          </label>
          <ImageFields
            v-model="imageFields"
            @selectedRowChange="onSelectedRowChange"
          ></ImageFields>
        </b-card-text>
      </b-card>
      Image preview
      <ImagePreview
        class="image_preview"
        :imageSrc="imageSrc"
        :imageFields="imageFields"
        :padding="padding"
        :focusedFieldIndex="focusedFieldIndex"
        :maxRooms="maxRooms"
      ></ImagePreview>
    </div>
    <div>
      <b-button variant="success" type="submit" class="mt-2">
        Save image
      </b-button>
    </div>
  </b-form>
</template>

<script>
import toastPresets from "@/utils/toastPresets.js";
import ImageFields from "@/components/displayContent/ImageFields.vue";
import ImagePreview from "@/components/displayContent/ImagePreview.vue";

export default {
  props: {
    initialImageFields: Array,
    initialPadding: Number,
    initialMaxRooms: Number,
    displayUuid: String,
    ignoreSchedule: Boolean,
  },
  components: {
    ImageFields,
    ImagePreview,
  },
  data() {
    return {
      selectedTemplateId: this.templateUuid,
      image: null,
      imageFields: this.initialImageFields || [],
      padding: this.initialPadding || 0,
      focusedFieldIndex: null,
      maxRooms: 1,
    };
  },
  computed: {
    templates() {
      return this.$store.state.templates;
    },
    imageSrc() {
      if (this.image) {
        return this.image;
      } else if (this.selectedTemplateId) {
        return `${this.$store.state.URI}/template/get-image/${
          this.selectedTemplateId
        }?x=${Date.now()}`;
      } else {
        return `${this.$store.state.URI}/display/get-image/${
          this.displayUuid
        }?x=${Date.now()}`;
      }
    },
  },
  mounted() {
    // let template = this.templates.find((t) => t.uuid === this.templateUuid);
    // this.maxRooms = template.maxRooms;
  },
  methods: {
    submitDisplayContent() {
      const { selectedTemplateId, padding, imageFields, image, displayUuid } =
        this;
      const data = {
        image,
        displayUuid,
        templateUuid: selectedTemplateId,
        displayContent: {
          imageFields,
          padding: Number(padding),
        },
      };

      this.$store
        .dispatch("updateDisplayContent", data)
        .then(() => {
          this.$bvToast.toast(
            "Display image saved successfully",
            toastPresets.successMessage
          );
        })
        .catch(() => {
          this.$bvToast.toast(
            "Failed to save image",
            toastPresets.errorMessage
          );
        });
    },
    onSelectedTemplateChange(value) {
      this.image = null;
      let template = this.templates.find((t) => t.uuid === value);
      this.imageFields =
        template.displayContent &&
        template.displayContent.imageFields.map((f) => {
          return { ...f };
        });
      this.padding = template.displayContent && template.displayContent.padding;
      this.maxRooms = template.maxRooms;
    },
    onIgnoreScheduleChange(checked) {
      this.$emit("onIgnoreScheduleChange", this.displayUuid, checked);
    },
    onSelectedRowChange(index) {
      this.focusedFieldIndex = index;
    },
  },
};
</script>

<style scoped>
.editor {
  overflow: auto;
}
.form_card {
  width: 60%;
  float: left;
  margin-bottom: 5px;
}
.form_group {
  text-align: left;
}
.image_preview {
  width: 38%;
}

/* Responsive layout - makes a one column-layout instead of two-column layout */
@media (max-width: 1200px) {
  .form_card {
    width: 100%;
  }
  .image_preview {
    width: 100%;
  }
}
</style>
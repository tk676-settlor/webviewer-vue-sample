<template>
  <button @click="submit">Submit</button>
  <div id="webviewer" ref="viewer"></div>
</template>

<script>
import {onMounted, ref} from 'vue';
import WebViewer from '@pdftron/webviewer';

export default {
  name: 'WebViewer',
  props: {initialDoc: {type: String}},
  setup(props) {
    const viewer = ref(null);
    const webViewerInstance = ref(null);

    onMounted(() => {
      const path = `${import.meta.env.BASE_URL}webviewer`;
      WebViewer({
        path,
        initialDoc: props.initialDoc,
        licenseKey: 'your_license_key'  // sign up to get a free trial key at https://dev.apryse.com
      }, viewer.value).then(
        (instance) => {
          const {documentViewer, annotationManager, Annotations} =
                instance.Core;
          webViewerInstance.value = instance;
          const fieldManager = annotationManager.getFieldManager();

          documentViewer.addEventListener('documentLoaded', () => {
            const field = new Annotations.Forms.Field(
              `test_sig`,
              {
                type: 'Sig',
              }
            );

            const signatureAnnot =
                  new Annotations.SignatureWidgetAnnotation(
                    field
                  );

            signatureAnnot.PageNumber = 1;
            signatureAnnot.X = 100;
            signatureAnnot.Y = 150;
            signatureAnnot.Width = 200;
            signatureAnnot.Height = 50;

            annotationManager.addAnnotation(signatureAnnot);
            fieldManager.addField(field);
            annotationManager.redrawAnnotation(signatureAnnot);
          });

        }
      );
    });

    function submit() {
      const {annotationManager} = webViewerInstance.value.Core;
      const allFields = annotationManager
        .getFieldManager()
        .getFields()
        .filter(
          (field) => field.widgets.length
        );
      console.log('allFields', allFields);
      for (const field of allFields) {
        console.log('field.widgets', field.widgets);
        const widget = field.widgets[0];
        const signature = extractAnnotationSignature(widget);
        console.log('signature', signature);
      }

    }

    function extractAnnotationSignature(annotation) {
      const {documentViewer} = webViewerInstance.value.Core;

      // Create a new Canvas to draw the Annotation on
      const canvas = document.createElement('canvas');
      // Reference the annotation from the Document
      const pageMatrix = documentViewer
        .getDocument()
        .getPageMatrix(annotation.PageNumber);

      // Set the height & width of the canvas to match the annotation
      canvas.height = annotation.Height;
      canvas.width = annotation.Width;
      // console.log('Canvas size:', canvas.width, canvas.height);

      const ctx = canvas.getContext('2d');
      // Translate the Annotation to the top Top Left Corner of the Canvas ie (0, 0)
      ctx.translate(-annotation.X, -annotation.Y);
      // console.log('Annotation coordinates:', annotation.X, annotation.Y);

      // Draw the Annotation onto the Canvas
      annotation.draw(ctx, pageMatrix);
      // console.log('Annotation drawn on canvas');

      canvas.toBlob((blob) => {
        // console.log("blob", blob, blob.text());
        const reader = new FileReader();
        reader.onload = () => {
          const dataUrl = reader.result;
          console.log('blob', blob, dataUrl);
          // Create an image element to visualize the blob
          const img = document.createElement('img');
          img.src = dataUrl;
          document.body.appendChild(img);
        };
        reader.readAsDataURL(blob);
      });

      const data = canvas.toDataURL('image/png');
      return data;
    }

    return {
      viewer,
      submit
    };
  },
};
</script>

<style>
#webviewer {
  height: 100vh;
}
</style>

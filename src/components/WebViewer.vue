<template>
  <div id="webviewer" ref="viewer"></div>
</template>

<script>
import { ref, onMounted } from 'vue';
import WebViewer from '@pdftron/webviewer';

export default {
  name: 'WebViewer',
  props: { initialDoc: { type: String } },
  setup(props) {
    const viewer = ref(null);
    onMounted(() => {
      const path = `${import.meta.env.BASE_URL}webviewer`;
      WebViewer({ 
        path, 
        initialDoc: props.initialDoc, 
        licenseKey: 'your_license_key'  // sign up to get a free trial key at https://dev.apryse.com
      }, viewer.value).then(
        (instance) => {
          const { documentViewer, annotationManager, Annotations } =
            instance.Core;

          documentViewer.addEventListener('documentLoaded', () => {
            const field = new Annotations.Forms.Field(
              `test_sig`,
              {
                type: "Sig",
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
            annotationManager.redrawAnnotation(signatureAnnot);
          });

          annotationManager.addEventListener(
            "annotationChanged",
            (annotation, action) => {
              console.log(
                "annotationManager annotationChanged",
                annotation,
                action
              );
            }
          );
        }
      );
    });
    return {
      viewer,
    };
  },
};
</script>

<style>
#webviewer {
  height: 100vh;
}
</style>

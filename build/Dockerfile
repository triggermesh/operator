FROM quay.io/operator-framework/helm-operator:v0.18.1

LABEL name="Triggermesh Operator" \
      vendor="Triggermesh" \
      version="v0.3.0" \
      release="1" \
      summary="The Triggermesh Operator for Openshift" \
      description="This is the Triggermesh operator which provides an interface to Knative and Tekton"

COPY ./licenses/* /licenses/

COPY helm-charts/ ${HOME}/helm-charts/
COPY watches.yaml ${HOME}/watches.yaml

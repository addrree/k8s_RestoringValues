pipeline {
  agent { label 'tool' }

  environment {
    PYTHONNOUSERSITE = "1"
    NAMESPACE        = "andreylab7"
    REGISTRY_ID      = "crp7njmi04tok4e72ohg"
    IMAGE            = "cr.yandex/${REGISTRY_ID}/restoringvalues:latest"
  }

  options {
    timestamps()
    timeout(time: 25, unit: 'MINUTES')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Fetch artifacts from L2') {
      steps {
        sh 'rm -rf deploy_art && mkdir -p deploy_art'
        copyArtifacts(projectName: 'AndreyIL/AndreyLAb2', selector: lastSuccessful())
        // если артефакты лежат в dist/ и корне — подстрой под твою L2
        sh '''
          set -e
          echo "Artifacts:"
          find . -maxdepth 3 -type f -name "*.tgz" -o -name "*.whl" | sed 's|^./||'
          mkdir -p deploy_art/dist
          cp -f app-restoringvalues.tgz deploy_art/ || true
          cp -f dist/*.whl deploy_art/dist/ || true
          ls -la deploy_art deploy_art/dist
        '''
      }
    }

    stage('Docker build') {
      steps {
        sh '''
          set -e
          docker build -t "$IMAGE" -f docker/Dockerfile .
        '''
      }
    }

    stage('Docker login + push to YCR') {
      steps {
        sh '''
          set -e
          docker login --username iam --password "$(yc iam create-token)" cr.yandex
          docker push "$IMAGE"
        '''
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          set -e
          kubectl get ns "$NAMESPACE" >/dev/null 2>&1 || kubectl create ns "$NAMESPACE"

          # подставляем registry_id в манифест (без helm)
          sed "s|cr.yandex/<REGISTRY_ID>/restoringvalues:latest|$IMAGE|g" k8s/deployment.yaml | kubectl apply -n "$NAMESPACE" -f -
          kubectl apply -n "$NAMESPACE" -f k8s/service-simulator.yaml
          kubectl apply -n "$NAMESPACE" -f k8s/service-reciever.yaml
          kubectl apply -n "$NAMESPACE" -f k8s/service-business.yaml

          kubectl rollout status deploy/restoringvalues -n "$NAMESPACE" --timeout=120s
          kubectl get pods -n "$NAMESPACE"
          kubectl get svc -n "$NAMESPACE"
        '''
      }
    }

    stage('Show logs') {
      steps {
        sh '''
          set -e
          POD=$(kubectl get pods -n "$NAMESPACE" -l app=restoringvalues -o jsonpath="{.items[0].metadata.name}")
          echo "POD=$POD"
          kubectl logs -n "$NAMESPACE" "$POD" --tail=200 || true
        '''
      }
    }
  }
}

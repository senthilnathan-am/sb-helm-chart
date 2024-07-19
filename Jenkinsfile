pipeline {
  agent any
  environment {
      branch_name="${Branch}"
      release_type="${Release}"
      AWS_ACCESS_KEY = credentials('7a5f353a-4c21-491f-a4a6-cb4738f5b0a9') 
      AWS_SECRET_KEY = credentials('590a8e9b-8854-431e-817c-08faef36799d')
      AWS_REGION = credentials('efcaf984-bf4c-4f34-a84c-5f0438bfcdba')
      AWS_ACCOUNT_ID = credentials('c620055a-b75b-40b2-a390-d780f977faa8')
  }

  stages {
    stage('Git Clone') {
      steps {
        script {
            git(url: 'https://git.assistanz.com/stackbill/sb-helm-charts.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: 'master')
            sh 'ls -al'
        }
      }
    }

    stage('Chart Version Update') {
        steps {
            sh '''
              aws ecr-public get-login-password --region us-east-1 | helm registry login --username AWS --password-stdin public.ecr.aws
              default_chart_version=$(grep -A3 description Chart.yaml | grep version | awk '{print $2}')
              old_chart_version=$(aws ecr-public describe-images --region us-east-1 --repository-name stackbill --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | awk 'NR==1{print}')
              old_app_version=`grep -i appversion Chart.yaml | awk '{print $2}' | tr -d '\"'`

              if [ "$branch_name" = "stable" ]; then
                old_core_image_tag=$(grep -A3 'sb-core' values.yaml | grep tag | awk '{print $2}' | tr -d '"')
                new_core_image_tag=$(aws ecr describe-images --repository-name stackbill-coreapi --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | grep -v "alpha" | grep -v "beta" | awk 'NR==1{print}')
                sed -i "/sb-core/{n;n;n;s/$old_core_image_tag/$new_core_image_tag/}" values.yaml
                new_app_version=$(echo $new_core_image_tag | tr -d v)
                sed -i "/appVersion/s/$old_app_version/$new_app_version/g" Chart.yaml

                old_billing_image_tag=$(grep -A3 'sb-billing' values.yaml | grep tag | awk '{print $2}' | tr -d '"')
                new_billing_image_tag=$(aws ecr describe-images --repository-name stackbill-billing --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | grep -v "alpha" | grep -v "beta" | awk 'NR==1{print}')
                sed -i "/sb-billing/{n;n;n;s/$old_core_image_tag/$new_core_image_tag/}" values.yaml

                if [ "$release_type" = "Major" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=0
                  k=0
                  i=$(expr $i + 1)
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                elif [ "$release_type" = "Minor" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f2`
                  k=0
                  if [ "$j" -gt 1000 ]; then
                    j=0
                    i=$(expr $i + 1)
                  else
                    j=$(expr $j + 1)
                  fi
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                elif [ "$release_type" = "Patch" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f2`
                  k=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f3`
                  if [ "$k" -gt 20 ]; then
                    exit;
                  else
                    k=$(expr $k + 1)
                  fi
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                fi
                if [ -f *.tgz ]; then
                  `rm -f *.tgz`
                fi
                helm package .
                helm push *.tgz oci://public.ecr.aws/p0g2c5k8
              fi

              if [ "$branch_name" = "development" ]; then
                old_core_image_tag=$(grep -A3 'sb-core' values.yaml | grep tag | awk '{print $2}' | tr -d '"')
                new_core_image_tag=$(aws ecr describe-images --repository-name stackbill-coreapi --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | tr -d 'v' | grep alpha | tr -d 'alpha"\"-' | awk 'NR==1{print}')
                sed -i "/sb-core/{n;n;n;s/$old_core_image_tag/$new_core_image_tag/}" values.yaml
                new_app_version=$(echo $new_core_image_tag | tr -d v)
                sed -i "/appVersion/s/$old_app_version/$new_app_version/g" Chart.yaml

                old_billing_image_tag=$(grep -A3 'sb-billing' values.yaml | grep tag | awk '{print $2}' | tr -d '"')
                new_billing_image_tag=$(aws ecr describe-images --repository-name stackbill-billing --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | tr -d 'v' | grep alpha | tr -d 'alpha"\"-' | awk 'NR==1{print}')
                sed -i "/sb-billing/{n;n;n;s/$old_core_image_tag/$new_core_image_tag/}" values.yaml

                if [ "$release_type" = "Major" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=0
                  k=0
                  i=$(expr $i + 1)
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                elif [ "$release_type" = "Minor" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f2`
                  k=0
                  if [ "$j" -gt 1000 ]; then
                    j=0
                    i=$(expr $i + 1)
                  else
                    j=$(expr $j + 1)
                  fi
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                elif [ "$release_type" = "Patch" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f2`
                  k=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f3`
                  if [ "$k" -gt 20 ]; then
                    exit;
                  else
                    k=$(expr $k + 1)
                  fi
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                fi
                if [ -f *.tgz ]; then
                  `rm -f *.tgz`
                fi
                helm package .
                helm push *.tgz oci://public.ecr.aws/p0g2c5k8
              fi

              if [ "$branch_name" = "pre-stable" ]; then
                old_core_image_tag=$(grep -A3 'sb-core' values.yaml | grep tag | awk '{print $2}' | tr -d '"')
                new_core_image_tag=$(aws ecr describe-images --repository-name stackbill-coreapi --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | tr -d 'v' | grep beta | tr -d 'beta"\"-' | awk 'NR==1{print}')
                sed -i "/sb-core/{n;n;n;s/$old_core_image_tag/$new_core_image_tag/}" values.yaml
                new_app_version=$(echo $new_core_image_tag | tr -d v)
                sed -i "/appVersion/s/$old_app_version/$new_app_version/g" Chart.yaml

                old_billing_image_tag=$(grep -A3 'sb-billing' values.yaml | grep tag | awk '{print $2}' | tr -d '"')
                new_billing_image_tag=$(aws ecr describe-images --repository-name stackbill-billing --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[*]' --output text | sort -r | tr -d '["\"]"\","\""' | tr -d 'v' | grep beta | tr -d 'beta"\"-' | awk 'NR==1{print}')
                sed -i "/sb-billing/{n;n;n;s/$old_core_image_tag/$new_core_image_tag/}" values.yaml

                if [ "$release_type" = "Major" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=0
                  k=0
                  i=$(expr $i + 1)
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                elif [ "$release_type" = "Minor" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f2`
                  k=0
                  if [ "$j" -gt 1000 ]; then
                    j=0
                    i=$(expr $i + 1)
                  else
                    j=$(expr $j + 1)
                  fi
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                elif [ "$release_type" = "Patch" ]; then
                  i=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f1`
                  j=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f2`
                  k=`echo $old_chart_version | awk "{print $1}" | cut -d "." -f3`
                  if [ "$k" -gt 20 ]; then
                    exit;
                  else
                    k=$(expr $k + 1)
                  fi
                  new_chart_version=$i.$j.$k
                  sed -i "/description/{n;n;n;s/$default_chart_version/$new_chart_version/}" Chart.yaml
                fi
                if [ -f *.tgz ]; then
                  `rm -f *.tgz`
                fi
                helm package .
                helm push *.tgz oci://public.ecr.aws/p0g2c5k8
              fi
            '''
        }
    }
  }
}
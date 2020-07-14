echo 'Logging in to AWS ECR...'
$(~/.local/bin/aws --profile ggcore ecr get-login --no-include-email --region ap-northeast-1)

echo 'Pulling image: build-%dep.GGCoreGgpcom_Build.build.number%'
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpcom_backoffice:build-%dep.GGCoreGgpcom_Build.build.number%
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpcom_widget:build-%dep.GGCoreGgpcom_Build.build.number%
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpcom_promotion:build-%dep.GGCoreGgpcom_Build.build.number%

echo 'Stopping all containers...'
docker ps --format '{{.Image}} {{.ID}}' | grep "ggpcom_" | awk '{print $2}' | xargs -I {} docker stop {}
# docker stop $(docker ps -aq)

echo 'Removing all containers...'
docker rm $(docker ps -aq)

echo 'Starting web services'
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPCOM -p 8011:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpcom_backoffice:build-%dep.GGCoreGgpcom_Build.build.number%
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPCOM -p 8012:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpcom_widget:build-%dep.GGCoreGgpcom_Build.build.number%
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPCOM -p 8019:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpcom_promotion:build-%dep.GGCoreGgpcom_Build.build.number%



echo 'Logging in to AWS ECR...'
$(~/.local/bin/aws --profile ggcore ecr get-login --no-include-email --region ap-northeast-1)

echo 'Pulling image: build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%'
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%

echo 'Stopping all containers...'
docker ps --format '{{.Image}} {{.ID}}' | grep "ggpokeruk_" | awk '{print $2}' | xargs -I {} docker stop {}

echo 'Removing all containers...'
docker rm $(docker ps -aq)

echo 'Starting web services'
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8014:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8020:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%

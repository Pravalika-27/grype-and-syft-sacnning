# grype-and-syft-sacnning

Scanning using grype and syft
yum install docker -y
systemctl start docker
sudo usermod -aG docker ec2-user
sudo chmod 666 /var/run/docker.sock



curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sudo sh -s -- -b /usr/local/bin
grype version
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sudo sh -s -- -b /usr/local/bin
syft --version

#pull images from docker hub:

envoyproxy/envoy - Docker Image | Docker Hub:docker pull envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e
envoyproxy/gateway - Docker Image | Docker Hub:docker pull envoyproxy/gateway:v1.4.5
envoyproxy/ratelimit - Docker Image | Docker Hub:docker pull envoyproxy/ratelimit:e74a664a


envoyproxy/ai-gateway-controller:docker pull envoyproxy/ai-gateway-controller:latest
envoyproxy/ai-gateway-cli:docker pull envoyproxy/ai-gateway-cli:latest

Generate sbom using syft:

syft envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e -o json > envoy_tools_dev_sbom.json
syft envoyproxy/ai-gateway-cli:latest -o json > ai_gateway_cli_sbom.json
syft envoyproxy/ai-gateway-controller:latest -o json > ai_gateway_controller_sbom.json
syft envoyproxy/gateway:v1.4.5 -o json > gateway_v1.4.5_sbom.json
syft envoyproxy/ratelimit:e74a664a -o json > ratelimit_sbom.json


Grype – Generate Vulnerability Reports:

grype envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e -o json > envoy_tools_dev_vuln.json
grype envoyproxy/ai-gateway-cli:latest -o json > ai_gateway_cli_vuln.json
grype envoyproxy/ai-gateway-controller:latest -o json > ai_gateway_controller_vuln.json
grype envoyproxy/gateway:v1.4.5 -o json > gateway_v1.4.5_vuln.json
grype envoyproxy/ratelimit:e74a664a -o json > ratelimit_vuln.json


Reports in tablular formate:

grype envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e -o table
grype envoyproxy/ai-gateway-cli:latest -o table
grype envoyproxy/ai-gateway-controller:latest -o table
grype envoyproxy/gateway:v1.4.5 -o table
grype envoyproxy/ratelimit:e74a664a -o table


#save table reports:
grype envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e -o table > envoy_tools_dev_vuln.txt
grype envoyproxy/ai-gateway-cli:latest -o table > ai_gateway_cli_vuln.txt
grype envoyproxy/ai-gateway-controller:latest -o table > ai_gateway_controller_vuln.txt
grype envoyproxy/gateway:v1.4.5 -o table > gateway_v1.4.5_vuln.txt
grype envoyproxy/ratelimit:e74a664a -o table > ratelimit_vuln.txt

For submission :
mkdir -p ~/container_security_reports/sbom_reports
mkdir -p ~/container_security_reports/grype_reports

mv *_sbom.json ~/container_security_reports/sbom_reports/
mv *_vuln.txt ~/container_security_reports/grype_reports/


#zip 
go to container_security folder 

zip -r grype_reports_txt.zip grype_reports/
zip -r grype_reports_json.zip grype_reports_json/
zip -r sbom_reports.zip sbom_reports/

To copy internally:

sudo cp /root/container_security_reports/grype_reports_json.zip /home/ec2-user/
sudo cp /root/container_security_reports/grype_reports_txt.zip /home/ec2-user/
sudo cp /root/container_security_reports/sbom_reports.zip /home/ec2-user/
sudo chown ec2-user:ec2-user grype_reports_txt.zip grype_reports_json.zip sbom_reports.zip
exit

 scp -i new-account.pem ec2-user@54.166.117.44:/home/ec2-user/*.zip .


####If no disk space:
docker system prune -a -f
docker volume prune -f
docker container prune -f
docker image prune -f


trivy scanning:

Trivy:
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin
trivy version

Step 2 — Basic Scan of Each Image
trivy image envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e
trivy image envoyproxy/ai-gateway-cli:latest
trivy image envoyproxy/ai-gateway-controller:latest
trivy image envoyproxy/gateway:v1.4.5
trivy image envoyproxy/ratelimit:e74a664a

Step 3 — Save Detailed Reports (JSON or HTML):
Automate Scanning for All Images:json

#!/bin/bash
mkdir -p trivy_reports
for image in \
  envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e \
  envoyproxy/ai-gateway-cli:latest \
  envoyproxy/ai-gateway-controller:latest \
  envoyproxy/gateway:v1.4.5 \
  envoyproxy/ratelimit:e74a664a
do
  safe_name=$(echo $image | tr '/:' '_')
  echo "🔍 Scanning $image..."
  trivy image -f json -o trivy_reports/${safe_name}.json $image
  echo "✅ Report saved to trivy_reports/${safe_name}.json"
done


To get html reports :
mkdir -p /usr/local/share/trivy/templates
wget -O /usr/local/share/trivy/templates/html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
ls /usr/local/share/trivy/templates/html.tpl

#!/bin/bash
mkdir -p trivy_reports/json trivy_reports/html

TEMPLATE_PATH="/usr/local/share/trivy/templates/html.tpl"
if [ ! -f "$TEMPLATE_PATH" ]; then
  echo "Downloading Trivy HTML template..."
  mkdir -p /usr/local/share/trivy/templates
  wget -q -O "$TEMPLATE_PATH" https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
fi

images=(
  "envoyproxy/envoy:tools-dev-c97cd05df9d5983873aee0e23b03f5ed7fad789e"
  "envoyproxy/ai-gateway-cli:latest"
  "envoyproxy/ai-gateway-controller:latest"
  "envoyproxy/gateway:v1.4.5"
  "envoyproxy/ratelimit:e74a664a"
)

for image in "${images[@]}"; do
  safe_name=$(echo "$image" | tr '/:' '_')
  echo "🔍 Scanning $image..."

  trivy image -f json -o "trivy_reports/json/${safe_name}.json" "$image"

  trivy image --format template --template "@${TEMPLATE_PATH}" \
    -o "trivy_reports/html/${safe_name}.html" "$image"

  echo "✅ Reports saved for $image"
done


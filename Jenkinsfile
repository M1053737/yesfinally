@Library("mule-4-shared-library") _
import com.mulesoft.PipelinePlaceholders

node {
    def pipelinePlaceholders = PipelinePlaceholders.getInstance()
    def jarFileName

    properties([
        parameters ([
            string(name: 'API_AUTO_DISCOVERY_ID', defaultValue: '', description: 'API Manager API Auto Discovery ID for Development Environment'),
            booleanParam(name: 'BUILD_RELEASE', defaultValue: false, description: 'Build a release')
        ])
    ])

    stage('Preparation') {
        deleteDir()
        checkout scm
        pipelinePlaceholders.setAuthorizationToken(getApAuthorizationToken())
        pipelinePlaceholders.setVaultSecrets(getVaultSecrets())
        pipelinePlaceholders.setApStructure(getApStructure())
        pipelinePlaceholders.setApiAutoDiscoveryId(params.API_AUTO_DISCOVERY_ID)
    }

    stage('Maven Build and Deploy') {
        jarFileName = mvnDeploy()
    }

    stage('Gather Development details') {
        pom = readMavenPom()
        pipelinePlaceholders.setMvnArtifactId(pom.getArtifactId())
        pipelinePlaceholders.setMvnGroupId(pom.getGroupId())
        pipelinePlaceholders.setMvnArtifactVersion(pom.getVersion())
        pipelinePlaceholders.setOrganization(pom.properties["organization"])
    }

    stage('Deploy to Development RTM') {
        muleChDeploy(jarFileName, "Development")
    }

    if(params.BUILD_RELEASE == true) {
        stage('Maven Prepare Release') {
            mvnPrepareRelease()
        }

        stage('Maven Release') {
            mvnRelease()
        }
    }

}
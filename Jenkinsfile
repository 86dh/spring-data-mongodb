def p = [:]
node {
	checkout scm
	p = readProperties interpolate: true, file: 'ci/pipeline.properties'
}

pipeline {
	agent none

	triggers {
		pollSCM 'H/10 * * * *'
		upstream(upstreamProjects: "spring-data-commons/main", threshold: hudson.model.Result.SUCCESS)
	}

	options {
		disableConcurrentBuilds()
		buildDiscarder(logRotator(numToKeepStr: '14'))
	}

	stages {
		stage("Docker images") {
			parallel {
				stage('Publish JDK (main) + MongoDB 4.0') {
					when {
						anyOf {
							changeset "ci/openjdk8-mongodb-4.0/**"
							changeset "ci/pipeline.properties"
						}
					}
					agent { label 'data' }
					options { timeout(time: 30, unit: 'MINUTES') }

					steps {
						script {
							def image = docker.build("springci/spring-data-with-mongodb-4.0:${p['java.main.tag']}", "--build-arg BASE=${p['docker.java.main.image']} --build-arg MONGODB=${p['docker.mongodb.4.0.version']} ci/openjdk8-mongodb-4.0/")
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								image.push()
							}
						}
					}
				}
				stage('Publish JDK (main) + MongoDB 4.4') {
					when {
						anyOf {
							changeset "ci/openjdk8-mongodb-4.4/**"
							changeset "ci/pipeline.properties"
						}
					}
					agent { label 'data' }
					options { timeout(time: 30, unit: 'MINUTES') }

					steps {
						script {
							def image = docker.build("springci/spring-data-with-mongodb-4.4:${p['java.main.tag']}", "--build-arg BASE=${p['docker.java.main.image']} --build-arg MONGODB=${p['docker.mongodb.4.4.version']} ci/openjdk8-mongodb-4.4/")
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								image.push()
							}
						}
					}
				}
				stage('Publish JDK (main) + MongoDB 5.0') {
					when {
						anyOf {
							changeset "ci/openjdk8-mongodb-5.0/**"
							changeset "ci/pipeline.properties"
						}
					}
					agent { label 'data' }
					options { timeout(time: 30, unit: 'MINUTES') }

					steps {
						script {
							def image = docker.build("springci/spring-data-with-mongodb-5.0:${p['java.main.tag']}", "--build-arg BASE=${p['docker.java.main.image']} --build-arg MONGODB=${p['docker.mongodb.5.0.version']} ci/openjdk8-mongodb-5.0/")
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								image.push()
							}
						}
					}
				}
				stage('Publish JDK (LTS) + MongoDB 4.4') {
					when {
						anyOf {
							changeset "ci/openjdk17-mongodb-4.4/**"
							changeset "ci/pipeline.properties"
						}
					}
					agent { label 'data' }
					options { timeout(time: 30, unit: 'MINUTES') }

					steps {
						script {
							def image = docker.build("springci/spring-data-with-mongodb-4.4:${p['java.lts.tag']}", "--build-arg BASE=${p['docker.java.lts.image']} --build-arg MONGODB=${p['docker.mongodb.4.4.version']} ci/openjdk17-mongodb-4.4/")
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								image.push()
							}
						}
					}
				}
			}
		}

		stage("test: baseline (main)") {
			when {
				beforeAgent(true)
				anyOf {
					branch(pattern: "main|(\\d\\.\\d\\.x)", comparator: "REGEXP")
					not { triggeredBy 'UpstreamCause' }
				}
			}
			agent {
				label 'data'
			}
			options { timeout(time: 30, unit: 'MINUTES') }
			environment {
				ARTIFACTORY = credentials("${p['artifactory.credentials']}")
			}
			steps {
				script {
					docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
						docker.image("harbor-repo.vmware.com/dockerhub-proxy-cache/springci/spring-data-with-mongodb-4.0:${p['java.main.tag']}").inside(p['docker.java.inside.basic']) {
							sh 'mkdir -p /tmp/mongodb/db /tmp/mongodb/log'
							sh 'mongod --setParameter transactionLifetimeLimitSeconds=90 --setParameter maxTransactionLockRequestTimeoutMillis=10000 --dbpath /tmp/mongodb/db --replSet rs0 --fork --logpath /tmp/mongodb/log/mongod.log &'
							sh 'sleep 10'
							sh 'mongo --eval "rs.initiate({_id: \'rs0\', members:[{_id: 0, host: \'127.0.0.1:27017\'}]});"'
							sh 'sleep 15'
							sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/tmp/jenkins-home" ./mvnw -s settings.xml clean dependency:list test -Duser.name=jenkins -Dsort -U -B'
						}
					}
				}
			}
		}

		stage("Test other configurations") {
			when {
				beforeAgent(true)
				allOf {
					branch(pattern: "main|(\\d\\.\\d\\.x)", comparator: "REGEXP")
					not { triggeredBy 'UpstreamCause' }
				}
			}
			parallel {
				stage("test: mongodb 4.4 (main)") {
					agent {
						label 'data'
					}
					options { timeout(time: 30, unit: 'MINUTES') }
					environment {
						ARTIFACTORY = credentials("${p['artifactory.credentials']}")
					}
					steps {
						script {
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								docker.image("harbor-repo.vmware.com/dockerhub-proxy-cache/springci/spring-data-with-mongodb-4.4:${p['java.main.tag']}").inside(p['docker.java.inside.basic']) {
									sh 'mkdir -p /tmp/mongodb/db /tmp/mongodb/log'
									sh 'mongod --setParameter transactionLifetimeLimitSeconds=90 --setParameter maxTransactionLockRequestTimeoutMillis=10000 --dbpath /tmp/mongodb/db --replSet rs0 --fork --logpath /tmp/mongodb/log/mongod.log &'
									sh 'sleep 10'
									sh 'mongo --eval "rs.initiate({_id: \'rs0\', members:[{_id: 0, host: \'127.0.0.1:27017\'}]});"'
									sh 'sleep 15'
									sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/tmp/jenkins-home" ./mvnw -s settings.xml clean dependency:list test -Duser.name=jenkins -Dsort -U -B'
								}
							}
						}
					}
				}

				stage("test: mongodb 5.0 (main)") {
					agent {
						label 'data'
					}
					options { timeout(time: 30, unit: 'MINUTES') }
					environment {
						ARTIFACTORY = credentials("${p['artifactory.credentials']}")
					}
					steps {
						script {
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								docker.image("harbor-repo.vmware.com/dockerhub-proxy-cache/springci/spring-data-with-mongodb-5.0:${p['java.main.tag']}").inside(p['docker.java.inside.basic']) {
									sh 'mkdir -p /tmp/mongodb/db /tmp/mongodb/log'
									sh 'mongod --setParameter transactionLifetimeLimitSeconds=90 --setParameter maxTransactionLockRequestTimeoutMillis=10000 --dbpath /tmp/mongodb/db --replSet rs0 --fork --logpath /tmp/mongodb/log/mongod.log &'
									sh 'sleep 10'
									sh 'mongo --eval "rs.initiate({_id: \'rs0\', members:[{_id: 0, host: \'127.0.0.1:27017\'}]});"'
									sh 'sleep 15'
									sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/tmp/jenkins-home" ./mvnw -s settings.xml clean dependency:list test -Duser.name=jenkins -Dsort -U -B'
								}
							}
						}
					}
				}

				stage("test: baseline (LTS)") {
					agent {
						label 'data'
					}
					options { timeout(time: 30, unit: 'MINUTES') }
					environment {
						ARTIFACTORY = credentials("${p['artifactory.credentials']}")
					}
					steps {
						script {
							docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
								docker.image("harbor-repo.vmware.com/dockerhub-proxy-cache/springci/spring-data-with-mongodb-4.4:${p['java.lts.tag']}").inside(p['docker.java.inside.basic']) {
									sh 'mkdir -p /tmp/mongodb/db /tmp/mongodb/log'
									sh 'mongod --setParameter transactionLifetimeLimitSeconds=90 --setParameter maxTransactionLockRequestTimeoutMillis=10000 --dbpath /tmp/mongodb/db --replSet rs0 --fork --logpath /tmp/mongodb/log/mongod.log &'
									sh 'sleep 10'
									sh 'mongo --eval "rs.initiate({_id: \'rs0\', members:[{_id: 0, host: \'127.0.0.1:27017\'}]});"'
									sh 'sleep 15'
									sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/tmp/jenkins-home" ./mvnw -s settings.xml clean dependency:list test -Duser.name=jenkins -Dsort -U -B'
								}
							}
						}
					}
				}
			}
		}

		stage('Release to artifactory') {
			when {
				beforeAgent(true)
				anyOf {
					branch(pattern: "main|(\\d\\.\\d\\.x)", comparator: "REGEXP")
					not { triggeredBy 'UpstreamCause' }
				}
			}
			agent {
				label 'data'
			}
			options { timeout(time: 20, unit: 'MINUTES') }

			environment {
				ARTIFACTORY = credentials("${p['artifactory.credentials']}")
			}

			steps {
				script {
					docker.withRegistry(p['docker.registry'], p['docker.credentials']) {
						docker.image(p['docker.java.main.image']).inside(p['docker.java.inside.basic']) {
							sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/tmp/jenkins-home" ./mvnw -v'
							sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/tmp/jenkins-home" ./mvnw -s settings.xml -Pci,artifactory ' +
									'-Dartifactory.server=https://repo.spring.io ' +
									"-Dartifactory.username=${ARTIFACTORY_USR} " +
									"-Dartifactory.password=${ARTIFACTORY_PSW} " +
									"-Dartifactory.staging-repository=libs-snapshot-local " +
									"-Dartifactory.build-name=spring-data-mongodb " +
									"-Dartifactory.build-number=${BUILD_NUMBER} " +
									'-Dmaven.test.skip=true clean deploy -U -B'
						}
					}
				}
			}
		}
	}

	post {
		changed {
			script {
				slackSend(
						color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
						channel: '#spring-data-dev',
						message: "${currentBuild.fullDisplayName} - `${currentBuild.currentResult}`\n${env.BUILD_URL}")
				emailext(
						subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
						mimeType: 'text/html',
						recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
						body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
			}
		}
	}
}

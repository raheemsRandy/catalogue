@Library("jenkins-shared-library")
 
def configMap = [
   // greeting : "Hello Jenkins"
   project:"roboshop",
   component:"catalogue"
]

if( ! env.BRANCH_NAME.equalsIgnoreCase('main')){
    nodejsEKSPipeline.call(configMap) // by default it will call,call funcn in this pipeline
}
else{
    echo "Please proceed with production process"
}

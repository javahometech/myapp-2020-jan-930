node{
   stage('scm checkout'){
     git 'https://github.com/vinodch501/myapp-2020-jan-930.git'
   }
   stage('compile-package'){
     sh 'mvn package'
   }
}

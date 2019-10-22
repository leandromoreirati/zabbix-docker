pipeline {
  //agent { label 'any' }
   stages {

      stage ('Deployment') {
          when { 
                branch 'develop' 
          }
        
         steps {
             sh '''
              echo "Hellow World!"
           '''
         }
       }

   }
}

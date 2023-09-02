def mul()
{
   def a=2
   def b=3
   def mul=a*b
   return mul
}

pipeline {
    agent any
    stages {
        stage('Hello') {
            steps{
                script{
                  output= mul()
                  echo "The mul is ${output}"
            }
            }
        }
    }
}

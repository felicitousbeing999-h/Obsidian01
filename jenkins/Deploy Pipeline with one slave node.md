![Pasted image 20260911200503.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911200503.png)![Pasted image 20260911200531.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911200531.png)![Pasted image 20260911200558.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911200558.png)![Pasted image 20260911205119.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911205119.png)


```groovy
pipeline {
    agent {label 'stapp01'}

    stages {
        stage('Deploy') {
            steps {
            sh '''
            cd /var/www/html/web_app
            git pull 
            cp -r /var/www/html/web_app/* /var/www/html/
            '''
            }
        }
    }
}

```


![Pasted image 20260911205541.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911205541.png)

or this approach simply if not ssh ing into the host stapp server application server anyway
```groovy
pipeline {
    agent {label 'stapp01'}

    stages {
        stage('Deploy') {
            steps {
            sh '''
            cd /var/www/html/
            git clone https://3000-port-dwiqf7khocgywmcq.labs.kodekloud.com/sarah/web_app.git /var/www/html/
            git pull 
            cp -r /web_app/*
            '''
            }
        }
    }
}

```





![Pasted image 20260911205801.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911205801.png)![Pasted image 20260911205840.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911205840.png)![Pasted image 20260911210254.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260911210254.png)
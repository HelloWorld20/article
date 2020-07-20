---
title: 部署脚本编写
date: 2017-06-08 09:20:07
tags: [vue, 前端技术]
---

# 前言

用Vue-cli搭建的脚手架使用的是webpack-dev-server搭建的服务器，一般情况下都是把开发版的代码另建一个文件，开发完成之后编译后在放到真正的java项目上。那么问题来了，每次打包webpack会在打包的文件名后添加hash值，每次部署都要修改一遍java入口文件里的script和link标签的引入文件名。还要把Vue项目的打包文件移动到java项目指定的位置。so，我觉得需要一个脚本，一键部署，即轻松还不易错。

<!-- more -->

下面贴代码

```javascript

const fs = require('fs')
const rimraf = require('rimraf')
const stat = fs.stat

const localConfig = {
    input: './dist',                                //打包文件路径
    output: '../CN201704A1/src/main/webapp/html',   //正式项目前端文件文件路径
    pages: '/index.html',                           //正式项目入口
    ignore: ['index.html']                          //不希望复制的文件名
}

inserFile(parseFile('index.html'))

function inserFile(fileData) {
    let path = localConfig.output + localConfig.pages

    let data = fs.readFileSync(path, 'utf-8')
    //先把字符转码一遍。
    let esData = escape(data)
    //正则替换掉四个引用的文件的hash值。
    esData = esData.replace(new RegExp( 'manifest\.[a-z0-9]{20}\.js'), 'manifest.' + fileData.manifest + '.js')
    esData = esData.replace(new RegExp( 'vendor\.[a-z0-9]{20}\.js'), 'vendor.' + fileData.vender + '.js')
    esData = esData.replace(new RegExp( 'app' + '\.[a-z0-9]{20}\.js'), 'app' + '.' + fileData.fileNameJs + '.js')
    esData = esData.replace(new RegExp( 'app' + '\.[a-z0-9]{32}\.css'), 'app' + '.' + fileData.fileNameCss + '.css')

    let result = unescape(esData)
    //删除原文件
    fs.unlinkSync(path)
    //写入新文件
    fs.writeFileSync(path, result)
    //删除指定文件夹，并写入新文件夹
    rimraf(localConfig.output + '/static', err => {
        if(err) throw new Erorr(err)
        exists( localConfig.input, localConfig.output, copy );
    })
}

function parseFile(fileName) {
    let fileData
    let path = localConfig.input + '/' + fileName;
    let data = fs.readFileSync(path, 'utf-8');

    let esData = escape(data);
    let shortFileName = fileName.slice(0, -5)

    let tempObj = {
        fileNameCss: esData.match(new RegExp( 'app' + '\.[a-z0-9]{32}\.css'))[0].slice(4, 36),
        fileNameJs: esData.match(new RegExp( 'app' + '\.[a-z0-9]{20}\.js'))[0].slice(4, 24),
        vender: esData.match(new RegExp( 'vendor' + '\.[a-z0-9]{20}\.js'))[0].slice(7, 27),
        manifest: esData.match(new RegExp('manifest\.[a-z0-9]{20}\.js'))[0].slice(9, 29)
    }
    console.log(tempObj)
    // fileData[shortFileName] = tempObj;

    return tempObj

}

/*
 * 复制目录中的所有文件包括子目录
 * @param{ String } 需要复制的目录
 * @param{ String } 复制到指定的目录
 */
function copy( src, dst ){
    // 读取目录中的所有文件/目录
    fs.readdir( src, function( err, paths ){
        if( err ){
            throw err;
        }

        paths.forEach(function( path ){
            var _src = src + '/' + path,
                _dst = dst + '/' + path,
                readable, writable;        

            if(localConfig.ignore.indexOf(path) != -1) return;

            stat( _src, function( err, st ){
                if( err ){
                    throw err;
                }
                // 判断是否为文件
                if( st.isFile() ){
                    // 创建读取流
                    readable = fs.createReadStream( _src );
                    // 创建写入流
                    writable = fs.createWriteStream( _dst );   
                    // 通过管道来传输流
                    readable.pipe( writable );
                }
                // 如果是目录则递归调用自身
                else if( st.isDirectory() ){
                    exists( _src, _dst, copy );
                }
            });
        });
    });
};

// 在复制目录前需要判断该目录是否存在，不存在需要先创建目录
function exists( src, dst, callback ){
    fs.exists( dst, function( exists ){
        // 已存在
        if( exists ){
            callback( src, dst );
        }
        // 不存在
        else{
            fs.mkdir( dst, function(){
                callback( src, dst );
            });
        }
    });
};

```

这是最简单的情况，只有一个入口且不用改变文件目录。如果是其他情况，还需要改造。
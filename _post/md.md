1.����mongocxxdriver
A. ������cdriver����
B.��װboostdev��cmake��
C.sudo cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DCMAKE_PREFIX_PATH=/usr/local -DLIBBSON_DIR=/usr/local -DBSONCXX_POLY_USE_BOOST=1
D. sudo make
�ڼ����Ҫ�޸Ĵ���

���Դ���test.cpp
#include <iostream>

#include <bsoncxx/builder/stream/document.hpp>
#include <bsoncxx/json.hpp>

#include <mongocxx/client.hpp>
#include <mongocxx/instance.hpp>

int main(int, char**) {
    mongocxx::instance inst{};
    mongocxx::client conn{mongocxx::uri{}};

    bsoncxx::builder::stream::document document{};

    auto collection = conn["testdb"]["testcollection"];
    document << "hello" << "world";

    collection.insert_one(document.view());
    auto cursor = collection.find({});

    for (auto&& doc : cursor) {
        std::cout << bsoncxx::to_json(doc) << std::endl;
    }
}

���뷽ʽ
c++ --std=c++11 test.cpp $(pkg-config --cflags --libs libmongocxx) -Wl,-rpath,/usr/local/lib




2. mongo ����
��װscons2.5.1����װssl,curl��pip2,python2.7
pip2 install -r buildscripts/requirements.txt
python2 buildscripts/scons.py all
# Cheatsheet


## Exercise 1

```sh
git clone https://github.com/AFLplusplus/AFLplusplus
cd AFLplusplus
make all
sudo make install
afl-fuzz --version
```

```sh
cd ..
wget http://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz
tar -xvf libpng-1.6.37.tar.gz
cd libpng-1.6.37/
CC=afl-clang-fast ./configure
make
```

```sh
mkdir inputs 
cp *.png inputs/
sudo afl-system-config
afl-fuzz -i inputs -o outputs -- ./.libs/pngstest @@
```

## Exercise 2

```sh
wget https://github.com/CodeIntelligenceTesting/jazzer/releases/download/v0.24.0/jazzer-linux.tar.gz
tar -xvf jazzer-linux.tar.gz
wget https://archive.apache.org/dist/commons/csv/binaries/commons-csv-1.9.0-bin.tar.gz
tar -xvf commons-csv-1.9.0-bin.tar.gz
cd commons-csv-1.9.0/
```

```sh
cat << 'EOF' > fuzzing.java
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import java.io.StringReader;

public class fuzzing {
    // Jazzer expects a public static method named fuzzerTestOneInput
    public static void fuzzerTestOneInput(byte[] data) {
        String input = new String(data);
        try {
            // Use the CSV parser with the default format
            CSVParser parser = CSVParser.parse(new StringReader(input), CSVFormat.DEFAULT);
            // Process the records; we're not using the output here
            parser.getRecords();
        } catch (Exception e) {
            // Catch and ignore exceptions: crashes are what Jazzer is looking for.
        }
    }
}
EOF
```

```sh
javac -cp commons-csv-1.9.0.jar -d out fuzzing.java
../jazzer --cp=./out:commons-csv-1.9.0.jar --target_class=fuzzing
```

## Exercise 3

```sh
mkdir jazzerjs
cd jazzerjs
npm init -y
npm install --save-dev @jazzer.js/core
```

```sh
npm install csv-parse

cat > FuzzTarget.js << 'EOF'
// FuzzTarget.js
const { parse } = require('csv-parse/sync');
module.exports.fuzz = function (data /*: Buffer */) {
  const input = data.toString('utf8');
  try {
    parse(input);
  } catch (err) {
    // Errors are caught here; Jazzer.js will report if something unexpected happens.
  }
};
EOF

npx jazzer FuzzTarget
```





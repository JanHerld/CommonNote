package com.cn.ustc.sqms.ctgs.file_opr;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.Properties;

import com.cn.ustc.sqms.ctgs.file_opr.myExceptions.MyFileCheckOldInterfaceException;

public class FileInformations {

	private String filepattern;
	private String logpath;
	private String filePath;

	/**
	 * Get file into <code>filelist</code> from specific directory derive from 
	 * <file>checkinfo.properties</file> 
	 * 
	 * @return
	 * @throws FileNotFoundException
	 * @throws IOException
	 */
	private ArrayList<File> propertiesGet() throws FileNotFoundException,
			IOException {
		String os = System.getProperty("os.name");
		Properties prop = new Properties();
		if (!os.equals("Windows 8")) {// check if the system ,if not windows 8 then default as server system.
			prop
					.load(new FileInputStream(
							"D:\\CommonFile\\DEVELOPMENT_SCRIPTS\\kettle_etl\\EDA\\properties\\checkinfo.properties"));
			this.logpath = prop.getProperty("logpath");
			this.filePath = prop.getProperty("filePath");
		} else {//load local properties,but the path loaded is just suitable for my own PC(zhang.huihua)
			prop
					.load(new FileInputStream(
							"F:\\CODE\\JAVA\\FileCheckDAPMkfjk\\src\\com\\cn\\ustc\\sqms\\ctgs\\file_opr\\config\\checkinfo.properties"));
			this.logpath = prop.getProperty("logpathLoc");
			this.filePath = prop.getProperty("filePathLoc");
		}
		this.filepattern = prop.getProperty("filePattern");
		System.out.println("文件目录：" + this.filePath + "\n正则表达式："
				+ this.filepattern);
		File[] files = null;
		ArrayList<File> filelist = new ArrayList<File>();
		if (!prop.isEmpty()) {
			File file = new File(filePath);
			if (file.isDirectory()) {
				files = file.listFiles();
				for (File f : files) {
					filelist.add(f);
				}
			}
			return filelist;
		}
		return null;
	}

	/**
	 * 
	 * @param fileName
	 * @throws FileNotFoundException
	 * @throws IOException
	 * @throws MyFileCheckOldInterfaceException
	 */
	public void filesMatchedGet(String fileName, Integer fieldNo,
			String dataDate) throws FileNotFoundException, IOException,
			MyFileCheckOldInterfaceException {
		ArrayList<File> filelist = propertiesGet();
		Iterator<File> itr = filelist.iterator();
		System.out.println("字段个数：" + fieldNo + "\t文件名：" + fileName);
		String[] strs = null;
		/*
		 * this variable aim to count the valid line-number in the file of
		 * specific interface. if <code>lineCount</code> equals to 0 when the
		 * procedure is over, then the file(<code>fileName</code>) is non-exist.
		 */
		Integer lineCount = 0;
		/*
		 * This is to store the fields of each lines split with sign '|',if its
		 * value is equal to <code>fieldNo</code> then continue,else throw
		 * wrong-checked exception
		 */
		Integer lineFieldsCount = 0;// 
		while (itr.hasNext()) {
			File fTemp = itr.next();
			/*
			 * Check Whether each file in <code>filelist</code> is matched.
			 */
			if (!fTemp.getName().matches(this.filepattern))
				filelist.remove(fTemp);// if false then remove.
		}
		System.out.println("过滤后的文件个数：" + filelist.size());
		for (File ft : filelist) {
			String temp = ft.getName();
			System.out.println("文件名称：" + temp);
			if (temp.matches(fileName)) {
				if (!ft.canRead()) {
					throw new MyFileCheckOldInterfaceException(
							"File required can't be read.", this.logpath);
				}
				FileReader fr = new FileReader(ft);
				BufferedReader br = new BufferedReader(fr);
				String recordLine = br.readLine();
				if (!recordLine.matches("START")) {
					throw new MyFileCheckOldInterfaceException(
							"There no File Starter sign in the file "
									+ fileName, this.logpath);
				} else {
					System.out.println("文件(" + fileName + ")开始标识符符合要求*^_^*");
				}
				while (recordLine != null) {
					recordLine = br.readLine();
					if (recordLine == null) {
						return;
					}
					strs = recordLine.split("\\|");
					if (lineCount == 0) {// Get Fields number of first line
						// record(not starter sign.).
						lineFieldsCount = strs.length;
					}
					if (!strs[0].equals("C::")) {
						if (strs[0].length() != 6)
							throw new MyFileCheckOldInterfaceException(
									"Length of data date is wrong(数据账期长度有误).",
									this.logpath);
						else {
							System.out.println("第" + (lineCount + 1)
									+ "行数据账期长度正确。*^_^*");
						}
						if (!strs[0].equals(dataDate))
							throw new MyFileCheckOldInterfaceException(
									"data date is wrong.(数据账期有误)", this.logpath);
						else {
							System.out.println("第" + (lineCount + 1)
									+ "数据账期正确*^_^*");
						}
						if (lineFieldsCount == strs.length) {
							if (strs.length != fieldNo) {
								throw new MyFileCheckOldInterfaceException(
										"Number of fields is not equal to the required in line "
												+ (lineCount + 1) + ".",
										this.logpath);
							}
							++lineCount;
						} else {
							throw new MyFileCheckOldInterfaceException(
									"Records' fields in the file " + fileName
											+ " are not equal,in number "
											+ lineCount + " line", this.logpath);
						}
					} else {
						if (!strs[0].equals("C::")
								&& !strs[1].equals(ft.getName())
								&& !strs[2].equals(lineCount)) {
							throw new MyFileCheckOldInterfaceException("File ("
									+ ft.getName()
									+ ") tail is not matched with its info.",
									this.logpath);
						} else {
							System.out.println("文件(" + fileName
									+ ")结束行内容符合要求*^_^*");
						}
					}
				}
			}
		}
		if (lineCount == 0) {
			throw new MyFileCheckOldInterfaceException(
					"File not exists which required.", this.logpath);
		}
	}

	/**
	 * This check program needs 3 values whose order are 1.file name which is
	 * going to be checked. 2.data lines of file,start sign and end sign
	 * exclusive. 3.data date of the interface,which should be complied with the
	 * form 'yyyymm'.
	 * 
	 * @param args
	 */
	public static void main(String[] args) {
		FileInformations fi = new FileInformations();
		try {
			fi.filesMatchedGet(args[0], Integer.parseInt(args[1]), args[2]);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (MyFileCheckOldInterfaceException e) {
			e.printStackTrace();
		}
	}

}

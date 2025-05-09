import React, { useState, useEffect } from "react";
import { useParams } from "react-router-dom";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

export default function CartaoFidelidadeApp() {
  const { lojistaParam, clienteParam } = useParams();
  const [lojista, setLojista] = useState("");
  const [cliente, setCliente] = useState("");
  const [pontos, setPontos] = useState({});
  const [logoPreview, setLogoPreview] = useState(null);
  const [configCartao, setConfigCartao] = useState({
    nomeCartao: "Cartão Fidelidade",
    corPrimaria: "#10b981",
    recompensa: "1 brinde gratuito",
    logo: null,
    linkCompartilhavel: ""
  });
  const totalPontos = 10;

  useEffect(() => {
    if (lojistaParam && clienteParam) {
      setLojista(decodeURIComponent(lojistaParam));
      setCliente(decodeURIComponent(clienteParam));
    }
  }, [lojistaParam, clienteParam]);

  useEffect(() => {
    if (lojista && cliente) {
      const link = gerarLinkCliente();
      setConfigCartao((prev) => ({ ...prev, linkCompartilhavel: link }));
    }
  }, [lojista, cliente]);

  const handleLogin = () => {
    if (!lojista) return;
  };

  const adicionarPonto = () => {
    if (!lojista || !cliente) return;
    setPontos((prev) => {
      const novosPontos = (prev[lojista]?.[cliente] || 0) + 1;
      const updated = {
        ...prev,
        [lojista]: {
          ...(prev[lojista] || {}),
          [cliente]: novosPontos,
        },
      };
      if (novosPontos >= totalPontos) {
        setTimeout(() => {
          alert(`🎉 ${configCartao.recompensa} conquistada!`);
        }, 100);
      }
      return updated;
    });
  };

  const gerarLinkCliente = () => {
    if (!lojista || !cliente) return "";
    const baseUrl = window.location.origin;
    const encodedLojista = encodeURIComponent(lojista);
    const encodedCliente = encodeURIComponent(cliente);
    return `${baseUrl}/fidelidade/${encodedLojista}/${encodedCliente}`;
  };

  const handleLogoUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setConfigCartao({ ...configCartao, logo: reader.result });
        setLogoPreview(reader.result);
      };
      reader.readAsDataURL(file);
    }
  };

  const pontosDoCliente = pontos[lojista]?.[cliente] || 0;

  if (lojistaParam && clienteParam) {
    return (
      <div className="p-4 max-w-xl mx-auto">
        <div className="flex items-center gap-4 mb-4">
          {configCartao.logo && (
            <img src={configCartao.logo} alt="Logo do Cartão" className="w-12 h-12 object-contain rounded" />
          )}
          <h1 className="text-2xl font-bold" style={{ color: configCartao.corPrimaria }}>
            {configCartao.nomeCartao} - {lojista}
          </h1>
        </div>
        <Card>
          <CardContent className="p-4">
            <h2 className="text-lg font-semibold mb-2">Cartão de {cliente}</h2>
            <div className="flex gap-2">
              {[...Array(totalPontos)].map((_, index) => (
                <div
                  key={index}
                  className={`w-6 h-6 rounded-full border-2 ${
                    index < pontosDoCliente
                      ? "border-green-500"
                      : "border-gray-300"
                  }`}
                  style={{ backgroundColor: index < pontosDoCliente ? configCartao.corPrimaria : "transparent", borderColor: index < pontosDoCliente ? configCartao.corPrimaria : "#d1d5db" }}
                ></div>
              ))}
            </div>
            {pontosDoCliente >= totalPontos && (
              <p className="mt-2 font-medium" style={{ color: configCartao.corPrimaria }}>
                🎉 {configCartao.recompensa} conquistada!
              </p>
            )}
          </CardContent>
        </Card>
      </div>
    );
  }

  if (!lojista) {
    return (
      <div className="p-4 max-w-xl mx-auto">
        <h1 className="text-2xl font-bold mb-4">Entrar como Lojista</h1>
        <Card className="mb-4">
          <CardContent className="p-4">
            <label className="block mb-2 font-medium">Nome do Lojista</label>
            <Input
              placeholder="Ex: Doce Amor"
              value={lojista}
              onChange={(e) => setLojista(e.target.value)}
            />
            <Button className="mt-4" onClick={handleLogin}>
              Entrar
            </Button>
          </CardContent>
        </Card>

        <Card>
          <CardContent className="p-4">
            <label className="block mb-2 font-medium">Nome do Cartão</label>
            <Input
              placeholder="Ex: Fidelidade Doce Amor"
              value={configCartao.nomeCartao}
              onChange={(e) => setConfigCartao({ ...configCartao, nomeCartao: e.target.value })}
            />

            <label className="block mt-4 mb-2 font-medium">Cor Primária</label>
            <Input
              type="color"
              value={configCartao.corPrimaria}
              onChange={(e) => setConfigCartao({ ...configCartao, corPrimaria: e.target.value })}
            />

            <label className="block mt-4 mb-2 font-medium">Recompensa</label>
            <Input
              placeholder="Ex: 1 brinde gratuito"
              value={configCartao.recompensa}
              onChange={(e) => setConfigCartao({ ...configCartao, recompensa: e.target.value })}
            />

            <label className="block mt-4 mb-2 font-medium">Logo do Cartão</label>
            <Input type="file" accept="image/*" onChange={handleLogoUpload} />
            {logoPreview && <img src={logoPreview} alt="Logo preview" className="mt-2 w-20 h-20 object-contain" />}

            {configCartao.linkCompartilhavel && (
              <div className="mt-4">
                <label className="block font-medium mb-1">Link Compartilhável do Cliente:</label>
                <Input value={configCartao.linkCompartilhavel} readOnly className="text-sm" />
              </div>
            )}
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="p-4 max-w-xl mx-auto">
      <div className="flex items-center gap-4 mb-4">
        {configCartao.logo && (
          <img src={configCartao.logo} alt="Logo do Cartão" className="w-12 h-12 object-contain rounded" />
        )}
        <h1 className="text-2xl font-bold" style={{ color: configCartao.corPrimaria }}>
          {configCartao.nomeCartao} - {lojista}
        </h1>
      </div>

      <Card className="mb-4">
        <CardContent className="p-4">
          <label className="block mb-2 font-medium">Telefone do Cliente</label>
          <Input
            placeholder="Ex: 11999999999"
            value={cliente}
            onChange={(e) => setCliente(e.target.value)}
          />
          <Button className="mt-4" onClick={adicionarPonto}>
            Adicionar Ponto
          </Button>

          {cliente && (
            <div className="mt-4">
              <label className="block font-medium mb-1">Link do Cliente:</label>
              <Input value={gerarLinkCliente()} readOnly className="text-sm" />
            </div>
          )}
        </CardContent>
      </Card>

      {cliente && (
        <Card>
          <CardContent className="p-4">
            <h2 className="text-lg font-semibold mb-2">
              Cartão de {cliente}
            </h2>
            <div className="flex gap-2">
              {[...Array(totalPontos)].map((_, index) => (
                <div
                  key={index}
                  className={`w-6 h-6 rounded-full border-2 ${
                    index < pontosDoCliente
                      ? "border-green-500"
                      : "border-gray-300"
                  }`}
                  style={{ backgroundColor: index < pontosDoCliente ? configCartao.corPrimaria : "transparent", borderColor: index < pontosDoCliente ? configCartao.corPrimaria : "#d1d5db" }}
                ></div>
              ))}
            </div>
            {pontosDoCliente >= totalPontos && (
              <p className="mt-2 font-medium" style={{ color: configCartao.corPrimaria }}>
                🎉 {configCartao.recompensa} conquistada!
              </p>
            )}
          </CardContent>
        </Card>
      )}
    </div>
  );
}
